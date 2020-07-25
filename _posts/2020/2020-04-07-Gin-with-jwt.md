---
layout: post
title: "Gin 框架使用JWT token认证"
date: "2020-04-07 12:00:00"
category: golang
tags: golang
author: Archer
---
* content
{:toc}

## 介绍

JWT全称JSON Web Token是一种跨域认证解决方案，属于一个开放的标准，它规定了一种Token实现方式，目前多用于前后端分离项目和OAuth2.0业务场景下。




## Gin 框架使用JWT token认证

### JWT是什么

JSON Web Token（缩写 JWT）是目前最流行的跨域认证解决方案

### 为什么JWT

Cookie-Session模式实现用户认证。相关流程大致如下：

1. 用户在浏览器端填写用户名和密码，并发送给服务端

2. 服务端对用户名和密码校验通过后会生成一份保存当前用户相关信息的session数据和一个与之对应的标识（通常称为session_id）

3. 服务端返回响应时将上一步的session_id写入用户浏览器的Cookie

4. 后续用户来自该浏览器的每次请求都会自动携带包含session_id的Cookie

5. 服务端通过请求中的session_id就能找到之前保存的该用户那份session数据，从而获取该用户的相关信息。

在分布式服务里，跨域认证中， session 数据持久化，工程量大，而JWT属于一种基于Token的轻量级认证模式，服务端认证通过后，会生成一个JSON对象，经过签名后得到一个Token（令牌）再发回给用户，用户后续请求只需要带上这个Token，服务端解密之后就能获取该用户的相关信息了

### JWT数据结构

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVc2VySUQiOjQsImV4cCI6MTU4NjcwNTMwMSwiaWF0IjoxNTg2MTAwNTAxLCJpc3MiOiJnaW5nbyIsInN1YiI6InVzZXIgdG9rZW4ifQ.y_192m4yUN7HPEkbtblMqxj5BFbyjA2hXCgFMdGxJI4
```

JWT 的三个部分依次如下:

- Header（头部）
- Payload（负载）
- Signature（签名）

#### Header

```shell
echo eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9 | base64 -d
```

解析查看得到：

```text
{"alg":"HS256","typ":"JWT"}
```

alg属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；typ属性表示这个令牌（token）的类型（type），JWT 令牌统一写为JWT。

#### Payload

JWT 规定了7个官方字段，供选用。

```text
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```

解析查看得到：

```text
{"UserID":4,"exp":1586705301,"iat":1586100501,"iss":"gingo","sub":"user token"}
```

其中`UserID`为自定义字段

#### Signature

需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256）

### 生成JWT和解析JWT

#### 定义claim

```go
// 服务端加密字符串
var jwtKey = []byte("jwt_key")

// Claims jwt，UserID为自定义字段，jwt.StandardClaims包含官方字段
type Claims struct {
	UserID uint
	jwt.StandardClaims
}
```

#### 生成JWT

```go
// ReleaseToken 发放token, model.User为自定义模型
func ReleaseToken(user model.User) (string, error) {
	expiresTime := time.Now().Add(7 * 24 * time.Hour)
	claims := &Claims{
		UserID: user.ID,
		StandardClaims: jwt.StandardClaims{
			ExpiresAt: expiresTime.Unix(),
			IssuedAt:  time.Now().Unix(),
			Issuer:    "gingo",
			Subject:   "user token",
		},
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	tokenString, err := token.SignedString(jwtKey)

	if err != nil {
		return "", err
	}

	return tokenString, nil
}
```

#### 解析JWT

```go
// ParseToken 解析token
func ParseToken(tokenString string) (*jwt.Token, *Claims, error) {
	claims := &Claims{}

	token, err := jwt.ParseWithClaims(tokenString, claims, func(token *jwt.Token) (i interface{}, err error) {
		return jwtKey, nil
	})

	return token, claims, err
}
```

### 在gin中使用JWT中间件

```go
// 定义需要认证的路由
r.GET("/api/auth/info", middleware.AuthMiddleware(), func(ctx *gin.Context) {
	user, _ := ctx.Get("user")
	ctx.JSON(200, gin.H{"user": user)
})
```

```go
// AuthMiddleware 认证
func AuthMiddleware()  gin.HandlerFunc{
	return func(ctx *gin.Context)  {
		// 获取 authorization hreader
		tokenString := ctx.GetHeader("Authorization")
		
		// 验证token格式
		if tokenString == "" || !strings.HasPrefix(tokenString, "Bearer "){
			ctx.JSON(http.StatusUnauthorized, gin.H{"code": 401, "message": "权限不足"})
			ctx.Abort()
			return
		} 

		tokenString = tokenString[7:]
		token, claims, err := common.ParseToken(tokenString)
		if err != nil || !token.Valid{
			ctx.JSON(http.StatusUnauthorized, gin.H{"code": 401, "message": "权限不足"})
			ctx.Abort()
			return
		}

		// 验证通过后获取claim中的userID
		userID := claims.UserID
		DB := common.GetDB()
		var user model.User
		DB.First(&user, userID)

		// 用户不存在
		if user.ID == 0 {
			ctx.JSON(http.StatusUnauthorized, gin.H{"code": 401, "message": "权限不足"})
			ctx.Abort()
			return
		}

		// 用户存在,存入上下文
		ctx.Set("user", user)
		ctx.Next()
	}
}
```