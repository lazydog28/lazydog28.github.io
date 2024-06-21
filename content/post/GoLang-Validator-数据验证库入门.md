---
title: "GoLang Validator 数据验证库入门"
date: 2023-07-29T21:16:14+08:00
tags:
  - golang
categories:
  - golang
---
> validator 开源仓库地址：https://github.com/go-playground/validator
> 
> 本文基于 validator v10.14.1 版本实现数据验证，自定义验证，自定义错误信息，错误信息翻译的实现。
>
> 才疏学浅，如有错误，欢迎指正。

# 1. 介绍与安装
## 1.1 介绍
Validator 是一个 Go 语言的开源库，用于对结构体字段进行验证。它支持许多常见的验证规则，例如必填、最小长度、最大长度、正则表达式等。使用 Validator 可以帮助开发者快速简洁地验证结构体字段，从而提高代码质量和安全性。

Validator 的使用非常简单，只需在结构体字段上添加验证规则即可。例如，以下代码验证 `Name` 字段必须是一个非空字符串：

```
type User struct {
    Name string `validate:"required"`
}
```

要验证结构体，只需调用 `validator.ValidateStruct()` 方法。例如，以下代码验证 `User` 结构体：

```
user := User{
    Name: "John Doe",
}

err := validator.ValidateStruct(&user)
if err != nil {
    fmt.Println(err)
}
```

如果 `User` 结构体是有效的，则 `ValidateStruct()` 方法不会返回任何错误。如果 `User` 结构体无效，则 `ValidateStruct()` 方法会返回一个包含错误信息的 `ValidationErrors` 结构体。

Validator 是一个非常强大的验证库，支持许多常见的验证规则。使用 Validator 可以帮助开发者快速简洁地验证结构体字段，从而提高代码质量和安全性。
## 1.2 安装
```shell
go get github.com/go-playground/validator/v10
```
# 2. 快速开始
## 2.1 基本使用
### 2.1.1 变量验证
```go
package main
import "github.com/go-playground/validator/v10"
var validate *validator.Validate
func main() {
	validate = validator.New()
	// 验证变量
	email := "joeybloggs.gmail.com"
	randomString := "1234567890"
	err := validate.Var(email, "required,email")
	if err != nil {
		println("email:" + err.Error())
	}
	err = validate.Var(randomString, "required,email")
	if err != nil {
		println("random" + err.Error())
	}
}
```
首先创建`validator`实例化对象，然后调用`validate.Var()`方法进行验证，第一个参数是需要验证的变量，第二个参数是验证规则，多个规则用逗号分隔。
如果不出意外，你会看到如下输出：
```shell
randomKey: '' Error:Field validation for '' failed on the 'email' tag
```
上述错误信息的意思是：`randomKey`字段验证失败，因为`randomKey`字段的值不是一个有效的邮箱地址。
#### 2.1.2 结构体验证
```go
package main

import "github.com/go-playground/validator/v10"

// 用户
type User struct {
	FirstName      string     `validate:"required"`
	LastName       string     `validate:"required"`
	Age            uint8      `validate:"gte=0,lte=130"`
	Email          string     `validate:"required,email"`
	FavouriteColor string     `validate:"iscolor"`                // alias for 'hexcolor|rgb|rgba|hsl|hsla'
	Addresses      []*Address `validate:"required,dive,required"` // a person can have a home and cottage...
}

// 地址
type Address struct {
	Street string `validate:"required"`
	City   string `validate:"required"`
	Planet string `validate:"required"`
	Phone  string `validate:"required"`
}

var validate *validator.Validate

func main() {
	validate = validator.New()
	user := User{}
	err := validate.Struct(user)
	if err != nil {
		println(err.Error())
	}
}
```
上文定义了结构体`Address`、`User`，其中`User`结构体包含了一个`Address`结构体的切片。`validate.Struct()`方法可以验证结构体，如果结构体中包含了其他结构体，那么需要使用`dive`关键字进行递归验证。

如果不出意外，你会看到如下输出：
```shell
Key: 'User.FirstName' Error:Field validation for 'FirstName' failed on the 'required' tag
Key: 'User.LastName' Error:Field validation for 'LastName' failed on the 'required' tag
Key: 'User.Email' Error:Field validation for 'Email' failed on the 'required' tag
Key: 'User.FavouriteColor' Error:Field validation for 'FavouriteColor' failed on the 'iscolor' tag
Key: 'User.Addresses' Error:Field validation for 'Addresses' failed on the 'required' tag
```
上述错误信息的意思是：`User`结构体中的`FirstName`字段验证失败，因为`FirstName`字段的值是一个空字符串。其他的错误信息也是类似的。
### 2.1.3 内置验证规则
#### Fields:

| Tag           | Description                                                 |
| ------------- | ----------------------------------------------------------- |
| eqcsfield     | Field Equals Another Field (relative)                       |
| eqfield       | Field Equals Another Field                                  |
| fieldcontains | Check the indicated characters are present in the Field     |
| fieldexcludes | Check the indicated characters are not present in the field |
| gtcsfield     | Field Greater Than Another Relative Field                   |
| gtecsfield    | Field Greater Than or Equal To Another Relative Field       |
| gtefield      | Field Greater Than or Equal To Another Field                |
| gtfield       | Field Greater Than Another Field                            |
| ltcsfield     | Less Than Another Relative Field                            |
| ltecsfield    | Less Than or Equal To Another Relative Field                |
| ltefield      | Less Than or Equal To Another Field                         |
| ltfield       | Less Than Another Field                                     |
| necsfield     | Field Does Not Equal Another Field (relative)               |
| nefield       | Field Does Not Equal Another Field                          |

#### Network:

| Tag              | Description                                 |
| ---------------- | ------------------------------------------- |
| cidr             | Classless Inter-Domain Routing CIDR         |
| cidrv4           | Classless Inter-Domain Routing CIDRv4       |
| cidrv6           | Classless Inter-Domain Routing CIDRv6       |
| datauri          | Data URL                                    |
| fqdn             | Full Qualified Domain Name (FQDN)           |
| hostname         | Hostname RFC 952                            |
| hostname_port    | HostPort                                    |
| hostname_rfc1123 | Hostname RFC 1123                           |
| ip               | Internet Protocol Address IP                |
| ip4_addr         | Internet Protocol Address IPv4              |
| ip6_addr         | Internet Protocol Address IPv6              |
| ip_addr          | Internet Protocol Address IP                |
| ipv4             | Internet Protocol Address IPv4              |
| ipv6             | Internet Protocol Address IPv6              |
| mac              | Media Access Control Address MAC            |
| tcp4_addr        | Transmission Control Protocol Address TCPv4 |
| tcp6_addr        | Transmission Control Protocol Address TCPv6 |
| tcp_addr         | Transmission Control Protocol Address TCP   |
| udp4_addr        | User Datagram Protocol Address UDPv4        |
| udp6_addr        | User Datagram Protocol Address UDPv6        |
| udp_addr         | User Datagram Protocol Address UDP          |
| unix_addr        | Unix domain socket end point Address        |
| uri              | URI String                                  |
| url              | URL String                                  |
| http_url         | HTTP URL String                             |
| url_encoded      | URL Encoded                                 |
| urn_rfc2141      | Urn RFC 2141 String                         |

#### Strings:

| Tag             | Description           |
| --------------- | --------------------- |
| alpha           | Alpha Only            |
| alphanum        | Alphanumeric          |
| alphanumunicode | Alphanumeric Unicode  |
| alphaunicode    | Alpha Unicode         |
| ascii           | ASCII                 |
| boolean         | Boolean               |
| contains        | Contains              |
| containsany     | Contains Any          |
| containsrune    | Contains Rune         |
| endsnotwith     | Ends Not With         |
| endswith        | Ends With             |
| excludes        | Excludes              |
| excludesall     | Excludes All          |
| excludesrune    | Excludes Rune         |
| lowercase       | Lowercase             |
| multibyte       | Multi-Byte Characters |
| number          | Number                |
| numeric         | Numeric               |
| printascii      | Printable ASCII       |
| startsnotwith   | Starts Not With       |
| startswith      | Starts With           |
| uppercase       | Uppercase             |

#### Format:

| Tag                           | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| base64                        | Base64 String                                                |
| base64url                     | Base64URL String                                             |
| base64rawurl                  | Base64RawURL String                                          |
| bic                           | Business Identifier Code (ISO 9362)                          |
| bcp47_language_tag            | Language tag (BCP 47)                                        |
| btc_addr                      | Bitcoin Address                                              |
| btc_addr_bech32               | Bitcoin Bech32 Address (segwit)                              |
| credit_card                   | Credit Card Number                                           |
| mongodb                       | MongoDB ObjectID                                             |
| cron                          | Cron                                                         |
| datetime                      | Datetime                                                     |
| e164                          | e164 formatted phone number                                  |
| email                         | E-mail String                                                |
| eth_addr                      | Ethereum Address                                             |
| hexadecimal                   | Hexadecimal String                                           |
| hexcolor                      | Hexcolor String                                              |
| hsl                           | HSL String                                                   |
| hsla                          | HSLA String                                                  |
| html                          | HTML Tags                                                    |
| html_encoded                  | HTML Encoded                                                 |
| isbn                          | International Standard Book Number                           |
| isbn10                        | International Standard Book Number 10                        |
| isbn13                        | International Standard Book Number 13                        |
| iso3166_1_alpha2              | Two-letter country code (ISO 3166-1 alpha-2)                 |
| iso3166_1_alpha3              | Three-letter country code (ISO 3166-1 alpha-3)               |
| iso3166_1_alpha_numeric       | Numeric country code (ISO 3166-1 numeric)                    |
| iso3166_2                     | Country subdivision code (ISO 3166-2)                        |
| iso4217                       | Currency code (ISO 4217)                                     |
| json                          | JSON                                                         |
| jwt                           | JSON Web Token (JWT)                                         |
| latitude                      | Latitude                                                     |
| longitude                     | Longitude                                                    |
| luhn_checksum                 | Luhn Algorithm Checksum (for strings and (u)int)             |
| postcode_iso3166_alpha2       | Postcode                                                     |
| postcode_iso3166_alpha2_field | Postcode                                                     |
| rgb                           | RGB String                                                   |
| rgba                          | RGBA String                                                  |
| ssn                           | Social Security Number SSN                                   |
| timezone                      | Timezone                                                     |
| uuid                          | Universally Unique Identifier UUID                           |
| uuid3                         | Universally Unique Identifier UUID v3                        |
| uuid3_rfc4122                 | Universally Unique Identifier UUID v3 RFC4122                |
| uuid4                         | Universally Unique Identifier UUID v4                        |
| uuid4_rfc4122                 | Universally Unique Identifier UUID v4 RFC4122                |
| uuid5                         | Universally Unique Identifier UUID v5                        |
| uuid5_rfc4122                 | Universally Unique Identifier UUID v5 RFC4122                |
| uuid_rfc4122                  | Universally Unique Identifier UUID RFC4122                   |
| md4                           | MD4 hash                                                     |
| md5                           | MD5 hash                                                     |
| sha256                        | SHA256 hash                                                  |
| sha384                        | SHA384 hash                                                  |
| sha512                        | SHA512 hash                                                  |
| ripemd128                     | RIPEMD-128 hash                                              |
| ripemd128                     | RIPEMD-160 hash                                              |
| tiger128                      | TIGER128 hash                                                |
| tiger160                      | TIGER160 hash                                                |
| tiger192                      | TIGER192 hash                                                |
| semver                        | Semantic Versioning 2.0.0                                    |
| ulid                          | Universally Unique Lexicographically Sortable Identifier ULID |
| cve                           | Common Vulnerabilities and Exposures Identifier (CVE id)     |

#### Comparisons:

| Tag            | Description             |
| -------------- | ----------------------- |
| eq             | Equals                  |
| eq_ignore_case | Equals ignoring case    |
| gt             | Greater than            |
| gte            | Greater than or equal   |
| lt             | Less Than               |
| lte            | Less Than or Equal      |
| ne             | Not Equal               |
| ne_ignore_case | Not Equal ignoring case |

### Other:

| Tag                  | Description          |
| -------------------- | -------------------- |
| dir                  | Existing Directory   |
| dirpath              | Directory Path       |
| file                 | Existing File        |
| filepath             | File Path            |
| image                | Image                |
| isdefault            | Is Default           |
| len                  | Length               |
| max                  | Maximum              |
| min                  | Minimum              |
| oneof                | One Of               |
| required             | Required             |
| required_if          | Required If          |
| required_unless      | Required Unless      |
| required_with        | Required With        |
| required_with_all    | Required With All    |
| required_without     | Required Without     |
| required_without_all | Required Without All |
| excluded_if          | Excluded If          |
| excluded_unless      | Excluded Unless      |
| excluded_with        | Excluded With        |
| excluded_with_all    | Excluded With All    |
| excluded_without     | Excluded Without     |
| excluded_without_all | Excluded Without All |
| unique               | Unique               |

##### Aliases:

| Tag          | Description                                                 |
| ------------ | ----------------------------------------------------------- |
| iscolor      | hexcolor\|rgb\|rgba\|hsl\|hsla                              |
| country_code | iso3166_1_alpha2\|iso3166_1_alpha3\|iso3166_1_alpha_numeric |

## 2.2 自定义验证器
```go
package main

import (
	"github.com/go-playground/validator/v10"
	"regexp"
)

var validate *validator.Validate

func IsPhone(fl validator.FieldLevel) bool {
	// 如果是空值则跳过验证
	if fl.Field().String() == "" {
		return true
	}
	// 正则验证手机号
	matched, _ := regexp.MatchString(`^1[3456789]\d{9}$`, fl.Field().String())
	return matched
}

func main() {
	validate = validator.New()
	err := validate.RegisterValidation("isphone", IsPhone)
	if err != nil {
		panic(err)
	}
	randomStr := "1234567890"
	err = validate.Var(randomStr, "isphone")
	if err != nil {
		println(err.Error())
	}
}
```
上文中的自定义验证器`IsPhone`，其实就是一个函数，函数的签名如下：
```go
func IsPhone(fl validator.FieldLevel) bool
```
当然，你也可以使用结构体的方法来实现自定义验证器，只需要将函数签名改为：
```go
func (s *Struct) IsPhone(fl validator.FieldLevel) bool
```
创建`validator`实例以后将自定义验证器注册进去即可：
```go
err := validate.RegisterValidation("isphone", IsPhone)
```

`RegisterValidation`方法接收两个参数，第一个参数是自定义验证器的名称，第二个参数是自定义验证器的实现，

注册成功后，就可以在验证器中使用了：

如果操作无误的话，你应该会看到如下输出：
```shell
Key: '' Error:Field validation for '' failed on the 'isphone' tag
```
## 2.3 翻译验证器错误信息
```go
package main

import (
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"
	zhTranslation "github.com/go-playground/validator/v10/translations/zh"
	"reflect"
	"regexp"
)

var validate *validator.Validate
var trans ut.Translator

func IsPhone(fl validator.FieldLevel) bool {
	// 如果是空值则跳过验证
	if fl.Field().String() == "" {
		return true
	}
	// 正则验证手机号
	matched, _ := regexp.MatchString(`^1[3456789]\d{9}$`, fl.Field().String())
	return matched
}
func initTranslate() {
	//初始化错误翻译器
	trans, _ = ut.New(zh.New()).GetTranslator("zh")
	validate = validator.New()
	//使用fld.Tag.Get("comment")注册一个获取tag的自定义方法以实现翻译Field
	validate.RegisterTagNameFunc(func(fld reflect.StructField) string {
		return fld.Tag.Get("tag")
	})
	err := validate.RegisterValidation("isphone", IsPhone)
	if err != nil {
		panic(err)
	}
	_ = validate.RegisterTranslation("isphone", trans, func(ut ut.Translator) error {
		return ut.Add("isphone", "{0}该参数不是一个手机号", true)
	}, func(ut ut.Translator, fe validator.FieldError) string {
		t, _ := ut.T("isphone", fe.Field())
		return t
	})
	err = zhTranslation.RegisterDefaultTranslations(validate, trans)
	if err != nil {
		println("注册错误翻译器失败：" + err.Error())
	}
	println("注册错误翻译器成功")
}

type User struct {
	Phone string `json:"phone" validate:"required,isphone" tag:"手机号"`
}

func main() {
	initTranslate()
	err := validate.Struct(User{
		Phone: "0123456789",
	})
	if err != nil {
		err := err.(validator.ValidationErrors)
		for _, e := range err.Translate(trans) {
			println(e)
		}
	}
}
```
上文中的`initTranslate`方法，主要是初始化错误翻译器，这里使用的是`zh`语言包，你也可以使用其他语言包，具体可以参考[这里](https://github.com/go-playground/validator/tree/master/translations)。
我们还是使用上面自定义验证器`isphone`，只不过这次我们将错误信息翻译成中文，你可以看到输出如下：
```shell
注册错误翻译器成功
手机号该参数不是一个手机号
```
注意翻译器翻译时不包括 `Field` 字段，因此我们需要在翻译器中添加 `{0}` 占位符，这样就可以将 `Field` 字段的值传入翻译器中了。

`RegisterTagNameFunc`函数的作用是注册一个获取tag的自定义方法以实现翻译Field，这里我们使用了`tag`作为`json`的别名，因此我们需要在`tag`中添加`tag`字段，这样就可以将`tag`字段的值传入翻译器中了。

至此，我们已经完成了验证器的基本使用，`gin`中使用验证器与上述一致。
