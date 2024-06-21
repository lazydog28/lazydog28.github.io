---
title: "Grpc 部署 ddddocr 验证码识别服务"
date: 2023-07-28T21:21:07+08:00
tags:
  - python
  - golang
  - 验证码识别
  - grpc
categories:
  - python
  - golang
---
> Python 带带弟弟 通用验证码识别OCR pypi版
>
> 开源仓库地址：https://github.com/sml2h3/ddddocr
> 
> 本文主要介绍如何使用python 将验证码识别服务部署到grpc上，方便其他语言调用，本文将使用golang调用python的grpc服务
> 
## 1. grpc简单介绍
grpc 是一个高性能、开源和通用的 RPC 框架，面向移动和 HTTP/2 设计，提供多种语言版本。

在 grpc 里客户端可以向调用本地对象一样直接调用另一台不同机器上服务端应用的方法，使得您能够更容易地创建分布式应用和服务。

与许多RPC系统类似，gRPC也是基于以下理念：定义一个服务，指定其能够被远程调用的方法（包含参数和返回类型）。在服务端实现这个接口。并运行一个gRPC服务器来处理客户端调用。在客户端拥有一个存根能够向服务端一样的方法。

HTTP/2采用二进制格式传输协议，而非HTTP/1.x的文本格式。grpc 消息使用 `protobuf` 编码。

`protobuf` 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC 数据交换格式。可以将其用于通信协议、数据存储等领域。

简而言之，grpc 更轻更快，但是不够灵活，普适性相对http较差，普遍用于内部服务调用，而不是对外提供服务。

## 2. ddddocr 简单介绍
ddddocr 是一个通用验证码识别OCR，支持多种验证码类型。支持ocr验证码识别、验证码目标检测、点选验证码识别、滑块验证码识别。

由于是通用验证码识别，所以识别率不是非常高（能用），但是可以通过训练提高识别率。

由于 ddddocr 是python编写，其他语言调用起来比较麻烦，所以本文将使用python将其部署到grpc上，方便其他语言调用。

下文将 ddddocr 的 classification、detection、slider 三个服务部署到grpc上。

## 3. grpc 服务端部署
### 3.1 安装依赖
```
grpcio
grpcio-tools
ddddocr>= 1.4.7
Pillow==9.5.0
```
笔者写这篇文章时 Pillow 的版本已经时10.0.0,Pillow，该版本与当前 ddddocr 1.4.7 版本不匹配，所以 Pillow 版本必须为 9.5.0，否则会报错。
`grpcio,grpcio-tools` 两个库则是 grpc 的依赖库。
```shell
pip install grpcio grpcio-tools ddddocr Pillow==9.5.0
```
### 3.2 定义 proto 文件
proto 文件定义了 grpc 服务的接口，以及接口的输入输出参数。

下文定义了 `DdddOcrService` 服务，其中有三个接口，分别是 `Classification`、`Detection`、`SlideComparison`，分别对应 ddddocr 的 `classification`、`detection`、`slider` 三个服务。

具体 proto 编写教程本文不过多赘述，可以参考 [grpc 官方文档](https://grpc.io/docs/languages/python/basics/)。

创建 `ddddocr.proto` 文件，内容如下：

```proto
syntax="proto3";


service DdddOcrService {
  rpc Classification (ImageRequest) returns (ClassificationResponse) {}
  rpc Detection (ImageRequest) returns (DetectionResponse) {}
  rpc SlideComparison(SlideComparisonRequest) returns (coordinate) {}
}


message ImageRequest {
  string imageBase64 = 1;
  bytes image = 2;
}

message ClassificationResponse {
  string code = 1;
}

message DetectionResponse {
  repeated rectangle poses  = 1;
}


message rectangle{
  coordinate leftTop = 1;
  coordinate rightBottom = 2;
}


message coordinate {
  uint32 x = 1;
  uint32 y = 2;
}

message SlideComparisonRequest {
   bytes targetBytes = 1;
   bytes backgroundBytes = 2;
}
```

生成 python 依赖代码
```shell
python -m grpc_tools.protoc -I . --python_out=. --pyi_out=. --grpc_python_out=. ./ddddocr.proto
```
生成的文件如下：
```shell
ddddocr_pb2.py # 数据类型
ddddocr_pb2.pyi # 数据类型接口
ddddocr_pb2_grpc.py # grpc 服务接口
```
### 3.3 实现 grpc 服务
创建 `ddddocr_server.py` 文件，内容如下：
```python
from concurrent import futures

import grpc
from ddddocr import DdddOcr, base64_to_image

from proto_file import dddocr_pb2
from proto_file import dddocr_pb2_grpc

ocr = DdddOcr(show_ad=False, ocr=True, beta=True)# beta可能效果更好，也可能较差
det = DdddOcr(show_ad=False, det=True)
slide = DdddOcr(show_ad=False, det=False, ocr=False)


class DdddOcrServiceServicer(dddocr_pb2_grpc.DdddOcrServiceServicer):
    def Classification(
            self, request: dddocr_pb2.ImageRequest, context
    ) -> dddocr_pb2.ClassificationResponse:
        if request.image:
            result = ocr.classification(request.image)
            return dddocr_pb2.ClassificationResponse(code=result)
        elif request.imageBase64:
            result = ocr.classification(request.imageBase64)
            return dddocr_pb2.ClassificationResponse(code=result)
        else:
            print("请求体异常")
            return dddocr_pb2.ClassificationResponse(code="error")

    def Detection(
            self, request: dddocr_pb2.ImageRequest, context
    ) -> dddocr_pb2.DetectionResponse:
        if request.imageBase64:
            request.image = base64_to_image(request.imageBase64)
        result = det.detection(request.image)
        poses = []
        for item in result:
            x_min, y_min, x_max, y_max = item
            leftTop = dddocr_pb2.coordinate(x_min, y_min)
            rightBottom = dddocr_pb2.coordinate(x_max, y_max)
            poses.append(dddocr_pb2.rectangle(leftTop=leftTop, rightBottom=rightBottom))
        return dddocr_pb2.DetectionResponse(poses=poses)

    def SlideComparison(self, request: dddocr_pb2.SlideComparisonRequest, context) -> dddocr_pb2.coordinate:
        targetBytes = request.targetBytes
        backgroundBytes = request.backgroundBytes
        result = slide.slide_comparison(targetBytes, backgroundBytes)
        target = result.get("target", [0, 0])
        start_x, start_y = target
        return dddocr_pb2.coordinate(x=start_x, y=start_y)


def serve():
    port = "50051"
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    dddocr_pb2_grpc.add_DdddOcrServiceServicer_to_server(
        DdddOcrServiceServicer(), server
    )
    server.add_insecure_port("[::]:" + port)
    server.start()
    print("Server started, listening on " + port)
    server.wait_for_termination()


if __name__ == "__main__":
    serve()
```

上文定义了 `DdddOcrServiceServicer` 类继承自生成的依赖文件`dddocr_pb2_grpc.py`中的`DdddOcrServiceServicer`类。
将类中的方法全部重写，以实现功能。

`serve` 函数用于启动 grpc 服务。

### 3.4 启动 grpc 服务
```shell
python ddddocr_server.py
```
启动成功后，会打印 `Server started, listening on 50051`，表示启动成功。

## 4. grpc 客户端调用
### 4.1 安装依赖
```shell
# Install the protocol compiler plugins for Go using the following commands:
go install google.golang.org/protobuf/cmd/protoc-gen-go
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
# 将 protoc-gen-go 和 protoc-gen-go-grpc 添加到 PATH 环境变量中
export PATH="$PATH:$(go env GOPATH)/bin"
```
### 4.2 生成 grpc 依赖代码
首先将上文的 `ddddocr.proto` 文件复制到 `proto` 目录下,在 proto 文件 `syntax="proto3";` 下一行添加 `option go_package = ".;ocr";` 然后执行以下命令：
```shell
protoc --go_out=. --go-grpc_out=. ./ddddocr.proto
```
`option go_package = ".;ocr";` 的作用是指定生成的文件的包名为 `ocr`。
生成的文件如下：
```shell
ddddocr.pb.go # 数据类型
ddddocr_grpc.pb.go # grpc 服务接口
```

### 4.3 编写 grpc 客户端
创建 `ddddocr_client.go` 文件，内容如下：
```go
package ddddocr

import (
	ocr "ddddocrGrpc/ddddocr/proto"
	"context"
	"encoding/base64"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"log"
)

var Base64ToByte = base64.StdEncoding.DecodeString

var (
	client ocr.DdddOcrServiceClient
)

func init() {
	// Set up a connection to the server.
	conn, err := grpc.Dial("127.0.0.1:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("did not connect: %v", err)
		return
	}
	client = ocr.NewDdddOcrServiceClient(conn)
}

func Classification(ctx context.Context, in *ocr.ImageRequest, opts ...grpc.CallOption) (*ocr.ClassificationResponse, error) {
	return client.Classification(ctx, in, opts...)
}
func Detection(ctx context.Context, in *ocr.ImageRequest, opts ...grpc.CallOption) (*ocr.DetectionResponse, error) {
	return client.Detection(ctx, in, opts...)
}
func SlideComparison(ctx context.Context, in *ocr.SlideComparisonRequest, opts ...grpc.CallOption) (*ocr.Coordinate, error) {
	return client.SlideComparison(ctx, in, opts...)
}
```
上文代码中的 `init` 函数用于初始化 grpc 客户端，`Classification`、`Detection`、`SlideComparison` 函数用于调用 grpc 服务。

至此，grpc 服务端和客户端已经编写完成，下面就可以使用 golang 调用 grpc 服务。