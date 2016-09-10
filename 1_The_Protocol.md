# RFC4511介绍

RFC4511规范描述LDAP协议内容，编码，是LDAP技术的主体部分

# 3 Protocol Model 协议模型

1. 客户端对服务器执行一个协议操作。在这个模型中，客户端发送一个描述操作的协议请求给服务器。服务器处理该请求，并返回该操作请求的回应信息。每一个操作的处理都是原子的。
2. 服务器需要返回在LDAP协议中定义的回应信息，但是这个操作不是必须同步的。换句话说，多个请求和回应并发的在服务器和客户端之间交换时，不一定是顺序完成的。当然，如果必须使用同步通信，那需要在客户端程序中进行控制。
3. LDAP核心协议的定义可以映射到X.500子集中，但不是一一对应的。

## 3.1 Operation and LDAP Message Layer Relationship

协议操作在LDAP message层进行交换。当链接关闭时，如果可以的话，任何未完成的操作都会被丢弃；如果不能丢弃，则寄去完成操作但不返回给客户端。同时，当连接关闭时，客户端**绝对不能**假设未完成的操作已经成功或失败。

# 4 Elements of Protocol 协议基础

LDAP协议使用ASN.1规范进行描述，使用ASN.1 BER编码规范进行传输。第5章详细说明了LDAP协议是如何编码和传输的。 为了支持未来对LDAP协议的扩展，对于尾部的无法识别的`SEQUENCE`标记，客户端和服务器**必须**忽略。

客户端通过`BindRequest`消息来表明自身的版本号，如果没有发送`Bind`，则服务器默认认为客户端的版本为version3或更高。

客户端通过读取`supportedLDAPVersion`消息的属性，来判断服务器服务器支持的协议版本。

## 4.1 Common Elements

本章节描述`LDAPMessage`封包格式

### 4.1.1 Message Envelope

下面是`LDAPMessage`格式定义：

```
LDAPMessage ::= SEQUENCE {
     messageID       MessageID,
     protocolOp      CHOICE {
          bindRequest           BindRequest,
          bindResponse          BindResponse,
          unbindRequest         UnbindRequest,
          searchRequest         SearchRequest,
          searchResEntry        SearchResultEntry,
          searchResDone         SearchResultDone,
          searchResRef          SearchResultReference,
          modifyRequest         ModifyRequest,
          modifyResponse        ModifyResponse,
          addRequest            AddRequest,
          addResponse           AddResponse,
          delRequest            DelRequest,
          delResponse           DelResponse,
          modDNRequest          ModifyDNRequest,
          modDNResponse         ModifyDNResponse,
          compareRequest        CompareRequest,
          compareResponse       CompareResponse,
          abandonRequest        AbandonRequest,
          extendedReq           ExtendedRequest,
          extendedResp          ExtendedResponse,
          ...,
          intermediateResponse  IntermediateResponse },
     controls       [0] Controls OPTIONAL }

MessageID ::= INTEGER (0 ..  maxInt)
```

如果服务器收到的`LDAPMessage`有问题，比如tag不识别，`messageID`不能解析，长度不对等，服务器_应该_返回`Notice of Disconnection`消息，并在`protocolError`中带上`resultCode`。同时，**必须** 立刻关闭LDAP会话。

在其他情况下，客户端或服务器不能解析LDAP PDU，_应该_ 断开LDAP会话。否则，服务器**必须**返回适当的错误信息。

#### 4.1.1.1 MessageID

请求中的`messageID`**必须**是一个非零整数，且在同一个会话中的每个请求的ID都不同。0被保留用于主动通知。一般情况下，`messageID`是一个自动的计数器。

客户端**不能**发送使用同样`messageID`的请求。除非能确定之前的请求已经完成。

### 基本的数据类型

文档中定义的N多的数据类型，比如LDAPSTRING，LDAPOID，LDAPDN等，都是从基本类型衍生出来的。这里就不一一记录了。不过结果消息`Result Message`还是记录一下吧。

```
LDAPResult ::= SEQUENCE {
     resultCode         ENUMERATED {
          success                      (0),
          operationsError              (1),
          protocolError                (2),
          timeLimitExceeded            (3),
          sizeLimitExceeded            (4),
          compareFalse                 (5),
          compareTrue                  (6),
          authMethodNotSupported       (7),
          strongerAuthRequired         (8),
               -- 9 reserved --
          referral                     (10),
          adminLimitExceeded           (11),
          unavailableCriticalExtension (12),
          confidentialityRequired      (13),
          saslBindInProgress           (14),
          noSuchAttribute              (16),
          undefinedAttributeType       (17),
          inappropriateMatching        (18),
          constraintViolation          (19),
          attributeOrValueExists       (20),
          invalidAttributeSyntax       (21),
                -- 22-31 unused --
          noSuchObject                 (32),
          aliasProblem                 (33),
          invalidDNSyntax              (34),
               -- 35 reserved for undefined isLeaf --
          aliasDereferencingProblem    (36),
               -- 37-47 unused --
          inappropriateAuthentication  (48),
          invalidCredentials           (49),
          insufficientAccessRights     (50),
          busy                         (51),
          unavailable                  (52),
          unwillingToPerform           (53),
          loopDetect                   (54),
               -- 55-63 unused --
          namingViolation              (64),
          objectClassViolation         (65),
          notAllowedOnNonLeaf          (66),
          notAllowedOnRDN              (67),
          entryAlreadyExists           (68),
          objectClassModsProhibited    (69),
               -- 70 reserved for CLDAP --
          affectsMultipleDSAs          (71),
               -- 72-79 unused --
          other                        (80),
          ...  },
     matchedDN          LDAPDN,
     diagnosticMessage  LDAPString,
     referral           [3] Referral OPTIONAL }
```

`resultCode`扩展定义在RFC4520的Section 3.8。返回码定义在附录A中。

如果服务器在一个操作请求中，检测到多个错误，只返回一个可以最好表示错误原因的错误码。

`diagnosticMessage`返回一些可读的诊断消息。这个消息的内容是非标准的，一般用于返回结果码的额外信息。如果没有，则为空。

## Other operations

章节4.2到章节4.14都是介绍具体的操作协议内容。太细节的东东了。如果需要具体的分析报文内容，再进行记录。

# 5 Protocol Encoding, Connection, and Transfer 协议编码，链接和传输

LDAP协议运行在TCP协议之上。协议分层如下：

```
            +----------------------+
            |  LDAP message layer  |
            +----------------------+ > LDAP PDUs
            +----------------------+ < data
            |      SASL layer      |
            +----------------------+ > SASL-protected data
            +----------------------+ < data
            |       TLS layer      |
Application +----------------------+ > TLS-protected data
------------+----------------------+ < data
  Transport | transport connection |
            +----------------------+
```

LDAP缺省使用389端口，如果使用SSL链接，则缺省使用636端口。当然，也可以使用其他有效的TCP端口

关闭LDAP会话，一般是客户端发送`UnbindRequest`消息，或服务器发送`Notice of Disconnection`通知。然后，停止`LDAP message layer`通信，关闭`SASL layer`，关闭`TLS layer`，最后断开TCP链接。

# 6 Security Considerations

LDAP协议可以使用明文密码做简单认证，也可以使用SASL机制认证。也允许使用证书方式进行认证。

`SASL`机制和TLS的认证方式在RFC4513文档中进行描述。

# 附录

- 附录A，返回码定义，非错误码
- 附录B，完整的LDAP协议内容格式
- 附录C，变更记录。
