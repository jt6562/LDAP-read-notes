# 1.介绍

RFC4513描述了LDAP协议的认证方法和安全机制。本文详细说明了如何通过`StartTLS`操作建立TLS链接的过程。

本文详细的说明了`simple bind`认证方法，包括匿名认证，未认证，名字/密码认证机制和基于SASL的认证方法。

客户端和服务器相互写作达成一些安全的功能。因此必须有一个最小的安全功能子集。

下面列举一下基本的安全风险，包括但不限于：

1. 未认证访问目录数据
2. 通过镜像其他用户的访问操作，非法访问目录数据
3. 通过镜像其他用户的访问操作，非法访问可重用的客户认证信息
4. 未认证修改目录数据
5. 未认证修改配置信息 6.

LDAP提供下列安全机制：

1. Bind操作认证。
2. 支持定制的ACL机制
3. 通过TLS或SASL机制的确保数据完整
4. 通过TLS或SASL机制的确保数据安全
5. 可配置的服务器资源访问控制
6. 通过TSL或SASL机制进行服务器认证

LDAP也可以被带外协议保护，比如IP安全层。

# 2.Implementation Requirements

LDAP服务器**必须**支持简单Bind方法的匿名认证机制。

LDAP**必须**支持Bind方法的名字/密码认证机制。并且**必须**有能力保护名字/密码认证信息，可以使用通过`StartTLS`操作建立的TLS链接。

当没有安全的数据服务器时，LDAP服务器_应该_不允许使用名字/密码认证机制。

支持TLS的LDAP实现**必须**支持`TLS_RSA_WITH_3DES_EDE_CBC_SHA`加密套件，_应该_支持`TLS_DHE_DSS_WITH_3DES_EDE_CBC_SHA`加密套件。推荐使用后者

# 3.StartTLS Operation

`StartTLS`操作在[RFC4511]的4.14章节定义，该操作用于建立一个TLS链接。使用TLS协议的目标是确保数据的完整和安全。

## 3.1.TLS Establishment Procedures

客户端发送`StartTLS`扩展请求，服务器回复，之后开始TLS协商过程(客户端发送ClientHello...)。

当服务器需要一个客户端证书，而客户端没有证书时，服务器使用一个本地安全策略决定是否成功完成TLS协商。

为了防止中间人攻击，客户端**必须**对服务器证书做校验。

在TLS链接建立后，客户端_应该_刷新所有服务器信息。避免中间人攻击。

## 3.2.Effect of TLS on Authorization State

TLS链接的建立，变更，关闭都会更改授权状态。

## 3.3.TLS Ciphersuites

# 4.Authorization State 授权状态

每一个LDAP会话都有一个授权状态。这个状态是由许多因素构成。一些因素可能受协议事件影响，比如Bind，StartTLS或TLS关闭；一些因素可能受外部因素影响，比如时间，服务器负载。

Bind操作允许客户端和服务器交换信息，以建立授权身份。Bind操作也可以用于将LDAP会话转移到匿名授权状态。

一旦LDAP会话建立，这个会话即拥有一个匿名授权身份。当收到Bind请求，服务器立即将会话改为匿名授权身份状态。如果Bind操作成功，会话将变为已请求认证状态。（也就是说，Bind操作是非幂等的。）

# 5.Bind Operation

Bind操作允许认证信息在客户端和服务器之间交换以建立新的授权状态。

## 5.1.Simple Authentication Method

Bind操作的简单认证方法提供三种认证机制：

- 匿名认证机制
- 未经过身份验证的验证机制
- 使用用户名(DN)/密码的认证机制

### 5.1.1.Anonymous Authentication Mechanism of Simple Bind

LDAP客户端可以通过发送一个名字长度为零，密码长度为零的Bind请求，明确地建立一个匿名授权状态。

### 5.1.2.Unauthenticated Authentication Mechanism of Simple Bind

LDAP客户端可以通过发送名字长度不为零，密码长度为零的一个Bind请求，来使用一个未经过身份验证的验证机制来建立一个匿名授权状态。

由客户端提供的DN名字的目的是用于跟踪(比如日志)。该名字不用来进行身份验证，不被用于授权目的。

这种认证机制有一个明显安全问题，当用户打算执行一个名字/密码认证时，无意中输入了一个空密码，这就会导致客户端实际执行的是未经过身份验证的验证机制。所以，客户端在名字/密码输入界面，应该不允许输入空密码。此外，服务器应该默认失败。

### 5.1.3.Name/Password Authentication Mechanism of Simple Bind

顾名思义，就是一个名字/密码认证机制。名字是DN，长度不为零。密码长度不为零。

如果发送的DN有语法错误，则返回码为`invalidDNSyntax`。如果DN语法正确，但不能用于认证或密码错误，则返回`invalidCredentials`

name为空，但密码不为空的情况，服务器行为未定义。

名字/密码认证机制不适合在非安全环境下使用。也就是说，如果要用这种认证机制，必须先确保客户端和服务器之间的通信的安全的。

## 5.2.SASL Authentication Method
