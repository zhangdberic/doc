# /oauth/token 获取token处理

作用：请求/oauth/token，来获取访问的token。

请求URL：/oauth/token

请求方法：POST

请求头：Authorization 值 Basic clientId:secret，使用base64处理

请求参数： grant_type授权类型，例如： grant_type=password 

​                     scope作用范围，例如： scope=webclient 

​                     username用户名，例如： username=john.carnell

​                     password密码，例如：password=password1

返回值：

{
    "access_token": "c59bae01-6c74-4367-899b-32ede6887e6d",
    "token_type": "bearer",
    "refresh_token": "2248b507-c88d-4fa1-9418-77560fdd8f6a",
    "expires_in": 3599,
    "scope": "webclient"
}

