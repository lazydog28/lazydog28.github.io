---
title: "Golang实现HTTP-FLV视频流下载"
date: 2023-07-16T15:26:01+08:00
tags:
  - http
  - 基础
categories:
  - golang
---
# Golang实现HTTP-FLV视频流下载

## flv数据格式

![](https://pic4.zhimg.com/v2-93df236b8bdeba4c8b2c57a3c466360f_r.jpg)

## 文件写入

```go
package download

import (
	"fmt"
	"io"
)

const ioRetryCount = 3

type FileWriter struct {
	file io.Writer
}

func NewFileWriter(file io.Writer) *FileWriter {
	return &FileWriter{file}
}

func (writer FileWriter) Write(b []byte) error {
	leftInputSize := len(b)
	for retryLeft := ioRetryCount; retryLeft > 0 && leftInputSize > 0; retryLeft-- {
		writtenCount, err := writer.file.Write(b[len(b)-leftInputSize:])
		leftInputSize -= writtenCount
		if err != nil {
			return err
		}
		if leftInputSize != 0 {
			fmt.Printf("doWrite() left %d bytes to write\n", leftInputSize)
		}
	}
	if leftInputSize != 0 {
		return fmt.Errorf("doWrite([%d]byte) tried %d times, but still has %d bytes to write", len(b), ioRetryCount, leftInputSize)
	}
	return nil
}

func (writer FileWriter) Close() error {
	if closer, ok := writer.file.(io.Closer); ok {
		return closer.Close()
	}
	return nil
}

func (writer FileWriter) Copy(r io.Reader, n int64) error {
	if writtenCount, err := io.CopyN(writer.file, r, n); err != nil || writtenCount != n {
		if err == nil {
			err = fmt.Errorf("doCopy(%d), %d bytes written", n, writtenCount)
		}
		return err
	}
	return nil
}

```

## 字节流读取

```go
package download

import (
	"errors"
	"io"
	"sync"
)

const defaultBufferSize = 1024

var pool = sync.Pool{New: func() interface{} { return make([]byte, defaultBufferSize) }}

func NewBufferedReader(rd io.ReadCloser) *BufferedReader {
	return &BufferedReader{
		ReadCloser: rd,
		buf:        pool.Get().([]byte),
		StopCh:     make(chan struct{}),
		closeOnce:  &sync.Once{},
	}
}

func (b *BufferedReader) ReadN(n int) ([]byte, error) {
	if n > len(b.buf)-b.r {
		println(len(b.buf), b.r, n)
		return nil, errors.New("n is bigger than len of buffer")
	}
	b.l = b.r
	return b.readN(n, b.l)
}

func (b *BufferedReader) readN(n, l int) ([]byte, error) {
	c, err := b.Read(b.buf[l : l+n])
	b.r += c
	if err != nil {
		return nil, err
	}
	if c < n {
		return b.readN(n-c, b.r)
	}
	return b.buf[b.l:b.r], nil
}

func (b *BufferedReader) ReadByte() (byte, error) {
	buf, err := b.ReadN(1)
	if err != nil {
		return 0, err
	}
	return buf[0], nil
}

func (b *BufferedReader) Reset() {
	b.l = 0
	b.r = 0
}

func (b *BufferedReader) Cap() int {
	return len(b.buf)
}

func (b *BufferedReader) AllBytes() []byte {
	return b.buf[:b.r]
}

func (b *BufferedReader) LastBytes() []byte {
	return b.buf[b.l:b.r]
}

func (b *BufferedReader) Free() {
	b.Reset()
	pool.Put(b.buf)
	b.buf = nil
}
```

## 解析字节流

```go
package download

import (
	"bytes"
	"encoding/binary"
	"errors"
	"io"
	"sync"
)

// FLV 封装格式解析 https://zhuanlan.zhihu.com/p/611128149

type BufferedReader struct {
	io.ReadCloser
	buf       []byte
	l, r      int
	StopCh    chan struct{}
	closeOnce *sync.Once
}

const (
	audioTag  uint8 = 8
	videoTag  uint8 = 9
	scriptTag uint8 = 18
	AAC       uint8 = 10
)

type TagHeader struct {
	TagType   uint8
	DataSize  uint32
	Timestamp uint32
	Header    []byte
}

func (body *BufferedReader) Stop() {
	body.closeOnce.Do(func() {
		close(body.StopCh)
		err := body.Close()
		if err != nil {
			return
		}
		body.Reset()
		pool.Put(body.buf)
		body.buf = nil
	})
}

func (body *BufferedReader) ParseHeader() (meta Metadata, header []byte, err error) {
	// 读取flv头部
	header, err = body.ReadN(9)
	if err != nil {
		panic(err)
	}
	// signature
	flvSign := []byte{0x46, 0x4c, 0x56, 0x01} // flv version01
	if !bytes.Equal(header[:4], flvSign) {
		allBody, err := io.ReadAll(body)
		allBody = append(header, allBody...)
		if err != nil {
			panic(err)
		}
		println(string(allBody))
		return meta, header, errors.New("flv 头部签名不正确")
	}
	// flv flag
	// https://pic4.zhimg.com/v2-93df236b8bdeba4c8b2c57a3c466360f_r.jpg
	meta = Metadata{}
	meta.HasVideo = uint8(header[4])&(1<<2) != 0
	meta.HasAudio = uint8(header[4])&1 != 0
	// offset must be 9
	if binary.BigEndian.Uint32(header[5:]) != 9 {
		return meta, header, errors.New("flv 头部大小不为9")
	}
	body.Reset()
	return meta, header, nil
}

func (body *BufferedReader) ParseTagHeader() (tag TagHeader, err error) {
	// 读取flv tag header
	tagByte, err := body.ReadN(15)
	if err != nil {
		return
	}
	tag.TagType = tagByte[4]
	tag.DataSize = uint32(tagByte[5])<<16 | uint32(tagByte[6])<<8 | uint32(tagByte[7])
	tag.Timestamp = uint32(tagByte[8])<<16 | uint32(tagByte[9])<<8 | uint32(tagByte[10]) | uint32(tagByte[11])<<24
	tag.Header = tagByte
	body.Reset()
	return
}

```

## 下载

```go
package download

import (
	"encoding/json"
	"errors"
	"net/http"
	"net/url"
	"time"
)

type Metadata struct {
	HasVideo, HasAudio bool
}

func ConnectFlvServer(url *url.URL) (*BufferedReader, error) {
	//var client = http.Client{Transport: &http3.RoundTripper{
	//	TLSClientConfig: &tls.Config{
	//		NextProtos: []string{"h3-46", "h3-43"}, // 设置支持的QUIC版本
	//	},
	//}}
	//var client = http.Client{Transport: &http2.Transport{
	//	TLSClientConfig: &tls.Config{
	//		NextProtos: []string{"h3-46", "h3-43"}, // 设置支持的QUIC版本
	//	},
	//}}
	var client = http.Client{}
	req, err := http.NewRequest("GET", url.String(), nil)
	if err != nil {
		panic(err)
	}
	//req.Header.Add("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36 Edg/114.0.1823.79:")
	req.Header.Add("User-Agent", "Chrome/59.0.3071.115")
	resp, err := client.Do(req)
	headerByte, err := json.Marshal(resp.Header)
	println(string(headerByte))
	if resp.Header.Get("Content-Length") != "" {
		println("Content-Length:" + resp.Header.Get("Content-Length"))
		return nil, errors.New("Content-Length not support")
	}
	if err != nil {
		panic(err)
	}
	return NewBufferedReader(resp.Body), nil
}

func DownLoad(reader *BufferedReader, writer *FileWriter) error {
	_, header, err := reader.ParseHeader()
	if err != nil {
		return err
	}
	err = writer.Write(header)
	if err != nil {
		return err
	}
	for {
		select {
		case <-reader.StopCh:
			println("stop:" + time.Now().String())
			return nil
		default:
			tag, err := reader.ParseTagHeader()
			if err != nil {
				panic(err)
			}
			err = writer.Write(tag.Header)
			err = writer.Copy(reader, int64(tag.DataSize))
			if err != nil {
				return err
			}
		}
	}

}

```

