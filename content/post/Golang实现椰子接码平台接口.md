---
title: "Golang实现椰子接码平台接口"
date: 2023-07-18T00:34:39+08:00
tags:
  - http
  - 基础
categories:
  - golang
---

> 椰子云API接口文档 :https://note.youdao.com/ynoteshare/index.html?id=ffed1add2492b7c37038758fe18fb83c&type=note&_time=1689611740385

## base 基础工具函数

```go
package codeReceiver

import (
	"io"
	"net/http"
)

type CodeReceiver struct {
	Host     string `json:"host"`
	UserName string `json:"username"`
	Password string `json:"password"`
	Sid      string `json:"sid" name:"项目id"`
	Author   string `json:"kid" name:"作者"`
	Token    string `json:"token" name:"登录凭证"`
}

var client = &http.Client{}

func (c *CodeReceiver) Login() (err error)        { return }
func (c *CodeReceiver) GetMobile() (phone string) { return }

func (c *CodeReceiver) get(inter string, params map[string]string) (res *http.Response, err error) {
	inter = splicingUrl(inter, params)
	req, err := http.NewRequest("GET", inter, nil)
	if err != nil {
		return
	}
	for i := 0; i < RetryTimes; i++ {
		res, err = client.Do(req)
		if err != nil {
			continue
		}
		return res, nil
	}
	return
}

func splicingUrl(inter string, params map[string]string) string {
	// 拼接 inter 和 params
	inter = inter + "?"
	for k, v := range params {
		inter = inter + k + "=" + v + "&"
	}
	// 去除最后一个 &
	inter = inter[:len(inter)-1]
	return inter
}
func closeBody(body io.ReadCloser) {
	err := body.Close()
	if err != nil {
	}
}
```
代码中的CodeReceiver是一个基类，其他所有平台的接码都继承这个类，这样可以减少代码量，方便维护。
其他平台的接码类只需要实现它的函数即可。
基本上现在所有的平台请求数据都是GET请求，虽然有的是POST请求，但是通常也支持GET请求，所以这里就只写了GET方法，如果有拓展需要的就再加上POST即可。
## 主体部分
```go
package codeReceiver

import (
	"errors"
	"github.com/tidwall/gjson"
	"io"
	"github.com/sirupsen/logrus"
)
const (
	RetryTimes = 3
)

var (
	logger = logrus.New()
)
type Yezi struct {
	CodeReceiver
}

func NewYezi(userName, passWord, sid string) (y *Yezi) {
	return &Yezi{
		CodeReceiver: CodeReceiver{
			Host:     "http://api.sqhyw.net:90",
			Author:   "795226",
			Sid:      sid,
			UserName: userName,
			Password: passWord,
		},
	}

}

func (y *Yezi) Login() (err error) {
	inter := y.Host + "/api/logins"
	params := map[string]string{
		"username": y.UserName,
		"password": y.Password,
	}
	res, err := y.get(inter, params)
	if err != nil {
		return
	}
	defer closeBody(res.Body)
	body, err := io.ReadAll(res.Body)
	if err != nil {
		return
	}
	y.Token = gjson.GetBytes(body, "token").String()
	message := gjson.GetBytes(body, "message").String()
	if y.Token == "" {
		err = errors.New(message)
		return
	}
	return
}

func (y *Yezi) GetMobile() (mobile string, err error) {
	inter := y.Host + "/api/get_mobile"
	params := map[string]string{
		"token":      y.Token,
		"project_id": y.Sid,
		"api_id":     y.Author,
	}
	res, err := y.get(inter, params)
	if err != nil {
		return
	}
	defer closeBody(res.Body)
	body, err := io.ReadAll(res.Body)
	mobile = gjson.GetBytes(body, "mobile").String()
	message := gjson.GetBytes(body, "message").String()
	if mobile == "" {
		return "", errors.New(message)
	}
	return
}
func (y *Yezi) GetSpecificMobile(mobile string) (err error) {
	inter := y.Host + "/api/get_mobile"
	params := map[string]string{
		"token":      y.Token,
		"project_id": y.Sid,
		"api_id":     y.Author,
		"phone_num":  mobile,
	}
	res, err := y.get(inter, params)
	if err != nil {
		return
	}
	defer closeBody(res.Body)
	body, err := io.ReadAll(res.Body)
	mobile = gjson.GetBytes(body, "mobile").String()
	message := gjson.GetBytes(body, "message").String()
	if mobile == "" {
		return errors.New(message)
	}
	return
}

func (y *Yezi) GetMessage(mobile string) (code string, err error) {
	inter := y.Host + "/api/get_message"
	params := map[string]string{
		"token":      y.Token,
		"project_id": y.Sid,
		"phone_num":  mobile,
	}
	res, err := y.get(inter, params)
	if err != nil {
		return
	}
	defer closeBody(res.Body)
	body, err := io.ReadAll(res.Body)
	if err != nil {
		return
	}
	code = gjson.GetBytes(body, "code").String()
	message := gjson.GetBytes(body, "message").String()
	if code == "" && message != "ok" {
		return "", errors.New(message)
	} else if code == "" {
		return "", errors.New("no message")
	}
	return
}

func (y *Yezi) AddBlackList(mobile string) (err error) {
	inter := y.Host + "/api/add_blacklist"
	params := map[string]string{
		"token":      y.Token,
		"project_id": y.Sid,
		"phone_num":  mobile,
	}
	res, err := y.get(inter, params)
	if err != nil {
		return
	}
	defer closeBody(res.Body)
	body, err := io.ReadAll(res.Body)
	message := gjson.GetBytes(body, "message").String()
	if message != "ok" {
		return errors.New(message)
	}
	return
}

```
通常用得上的主要的函数如上所写，没啥技术性，仅做分享，不再赘述。