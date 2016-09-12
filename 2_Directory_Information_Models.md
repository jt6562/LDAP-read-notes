# 1\. RFC4512介绍

RFC4512规范描述了LDAP协议使用的X.500目录信息模型。

目录是一协议开放系统的集合起来，互相协作以提供目录服务。在目录中的信息被称为`Directory Information Base (DIB)`。一个目录用户(可能是个自然人或其他其他主体)，可以通过客户端或目录用户代理`Directory User Agent (DUA)`访问目录。客户端可以与一个或多个服务器`Directory System Agents (DSA)`交互。

DIB包含下面两类信息：

1. 用户信息。比如由用户提供和管理的信息。详见[第2节用户信息模型](#2-model-of-directory-user-information-用户信息模型)
2. 管理和操作信息。详见[第3节目录管理和操作模型](#3-directory-administrative-and-operational-information-目录管理和操作模型)

这两个信息模型描述信息在目录中是如何体现的。这两个信息模型也被称为通用目录信息模型，它们为其他信息模型提供了一个框架。[第4节](#4-directory-schema-目录模式)讨论了子模式信息模型和子模式发现。[第5节](#5-dsa-server-informational-model-服务器信息模型)讨论了DSA（Server）信息模型。

其他X.500信息模型，比如访问控制信息模型，也可以被适配后用于LDAP中。

## 其他

第一节后面的内容，主要是LDAP规范的更新记录，文法惯例，语法(ABNF[RFC4334])。这里略过。

# 2\. Model of Directory User Information 用户信息模型

按照X.501定义的:

> 目录是用来保存对象(object)信息，并提供访问功能。对象可以是任何能被识别的东东。

> 对象类(object class)是一个可识别的对象家族，它们有某个相同性质。一个对象至少属于一个对象类。一个对象类可以是其他对象类的子类。

目录条目或目录项`directory entry`是目录中的基本单元。有多种类型目录条目。

一个目录条目代表一个对象。别名条目提供对象的别名。子条目保存管理和操作信息。

一组条目表示一个有层级结构的DIB信息，使用树状结构保存。这个称为目录信息树`Directory Information Tree (DIT)`。

## 2.1\. The Directory Information Tree 目录信息树

DIB是一组条目集合，它们按照树状结构组织起来。树的顶点也是条目。

顶点之间的连线定义了条目之间的关系。如果有一条线从X到Y，则表示X的是Y的直接上级，Y是X的直接下级。

类似的，对象条目的上下级关系可以被用来确定对象直接的关系。DIT结构规则被用于管理对象之间的关系。（注：没看懂）

注：一个条目的直接上级也被认为是这个条目的父节点，直接下级是子节点。有同样父节点的条目是兄弟节点。

## 2.2\. Structure of an Entry 目录项结构

一个条目由一组包含对象信息的属性组成。一些表示用户信息的属性叫做用户属性。其他的表示操作和管理信息的属性称为操作属性。

一个属性由一个属性描述和相关的一个或多个关联值组成。

一个属性的类型决定了该属性是否可以有多个值，属性的语法，用于构造和比较的匹配规则，以及其他功能。

属性值遵照属性类型的语法定义。

一个属性如果有两个的值，不应该认为是相等的。当且仅当两个值有相同的匹配规则时，可以被认为相等。或者，如果属性类型没有定义匹配规则，两个值当且仅当两个值完全相同的时候才相等。

举个例子，一个`givenName`属性可以有多个值，它们应该是字符串类型，且大小写不敏感。那么，`givenName`不能同时持有`John`和`JOHN`，因为他们是相等的值。

当一个属性用于命名条目时，只有一个属性值可以被用来构造相对专有名字`Relative Distinguished Name (RDN)`。这个值叫做`distingushed value`。

## 2.3\. Naming of Entries 条目命名

### 2.3.1\. Relative Distinguished Names 相对专有名字

每一个条目都有一个相对它上级的名字。这个相对名字就叫做`RDN`。RDN由一组无需的多个属性断言`an unordered set of one or more attribute value assertions (AVA)`组成。

一个条目的RDN在他所有兄弟节点中必须是唯一的。

下面是一些RDN的例子：

```
UID=12345
OU=Engineering
CN=Kurt Zeilenga+L=Redwood Shores
```

最后一个例子是由多个AVA组成的多值RDN。

### 2.3.2\. Distinguished Names 专有名字

一个条目的全称叫做`DN`。有RDN和它的父节点组成。比如：

```
UID=nobody@example.com,DC=example,DC=com
CN=John Smith,OU=Sales,O=ACME Limited,L=Moab,ST=Utah,C=US
```

### 2.3.3\. Alias Names 别名

条目的别名。参见[Section 2.6](#alias-entries-条目别名)

## 2.4\. Object Classes 对象分类

对象分类一系列有相同性质的可识别的对象家族。在X.501定义如下：

> 在目录中，对象类型的作用有多种目的：

> - 描述和分类的对象和对应于这些对象的条目;
> - 在适当的地方，控制目录的操作;
> - 结合目录树结构规则规范，调整目录树中条目的位置;
> - 结合目录树内容规则规范，调整条目中的属性;
> - 标识要与相应管理权限的特定策略相关的条目类别。

> 一个对象类可以是从一个对象类或它的超类派生出来的，而这个对象类或超类是从另一个更通用的对象类派生出来的。对于结构化的对象类，这个(派生)过程将在顶层通用类停止。

> 一个对象类可以从两个或更多的直接超类派生出来。子类的这种特性被称为多重继承。

### 2.4.1\. Abstract Object Classes 抽象类

抽象类是一个基类，提供了一些基础的特性，其他类可以从抽象类继承。条目不能直接使用抽象类。

抽象类不能从结构类和辅助类派生。

所有结构性对象类都是从抽象类`top`直接或间接派生出来的。辅助类不必须从`top`类派生。

### 2.4.2\. Structural Object Classes 结构类

最常见的类型。

比如：`person`，`organizationUnit`

### 2.4.3\. Auxiliary Object Classes 辅助类

用于扩充条目性质。 比如：`extensibeObject`

辅助类不能是结构类的子类。

## 2.5\. Attribute Description 属性描述

一个属性描述是由一个属性类型和一组0个或多个属性选项组成。属性描述使用ABNF([RFC5234](https://zh.wikipedia.org/wiki/%E6%89%A9%E5%85%85%E5%B7%B4%E7%A7%91%E6%96%AF%E8%8C%83%E5%BC%8F))表示为：

```
attributedescription = attributetype options
attributetype = oid
options = *( SEMI option )
option = 1*keychar
```

其中，`attributetype`标示属性类型，每个`option`标示一个属性选项。`attributetype`和`option`都是不区分大小写的。`option`没有顺序要求。

一条属性描述如果包含无法识别的属性类型，则该属性描述被认为无法识别。服务器将含有无法识别属性选项的属性描述标记为不可识别。客户端可以把无法识别的属性选项作为[标记选项](#2521-tagging-options)

一个条目的所有属性必须具有不同的属性描述。

### 2.5.1\. Attribute Types 属性类型

属性类型决定属性是否可以有多个值，属性的语法，用于狗将和比较的匹配规则，以及其他功能。

如果属性类型没有指定等效匹配:

- 属性(类型)不能用于命名
- 当添加/替换属性，不能有两个值是相等的
- 一个多值属性的各个值不能单独的添加或删除
- 使用类型值的属性值断言`AVA`不能被执行。（注：不明觉厉，没有示例，看不太懂） 否则，指定等效匹配规则将被用于评估属性类型的属性值断言。这个指定的等效规则，要被传递和交换。

属性类型表明一个属性是用户属性还是操作属性。如果是操作属性，属性类型表示操作用法，以及属性是否可以被用户修改。操作属性细节在[章节3.4](#34-)

一个属性类型（子类型）可以从一个更通用的属性类型派生出来。以下限制适用于子类型:

- 一个子类型必须有和父类型相同的用法
- 一个子类型的语法必须和父类型相同，或是父类型语法的细化
- 如果一个子类型的父类型是集体性的，那么子类型也必须是集体性的

每个属性类型都是由对象标识符和可选的一个或多个短名识别。

### 2.5.2\. Attribute Options 属性选项

属性描述有多种选项。LDAP规范详细说明一种:标签选项。

不是所有的选项都能和目录中的属性建立关联。标签选项可以。

不是所有选项都能与所有属性结合。这种情况下，属性描述被认为不可识别。

一个属性描述中，如果包含互斥的选项，则属性被认为不可识别。比如，`cn;x=bar;x-foo`,其中`x-bar`和`x-foo`是互斥的，那这个属性被认为不可识别。

其他种类的选项可能在未来的文档中规定。这些文档必须详细说明新的选项与标签选项的关系。特别是，这些文档必须详细说明新的选项是否可以关联到属性中，新的选项如何影响属性值的转换，新的选项在属性描述层次中如何被处理。

选项表示要短，不区分大小写。

如果要注册新的选项，可以参考RFC4520。

#### 2.5.2.1\. Tagging Options

属性可以有个任意多个标签选项。标签选项是永远不会冲突的。

一个包含N个标签选项的属性描述是一个具有相同属性类型描述但只有N中的一个选项的属性描述的直接子类型。原文如下：

```
An attribute description with N tagging options is a direct (description) subtype of all attribute descriptions of the same attribute type and all but one of the N options.
```

如果一个属相类型有一个父类型，那该类型对应的属性描述也有一个直接的包含父类型和N个标签选项的属性描述。比如：

- `cn;lang-de;lang-en`是`cn;lang-de`的直接子类型，当然，也是`cn;lang-en`的直接子类型
- `cn;lang-de;lang-en`也是`name;lang-de;lang-en`的直接子类型。因为`cn`是`name`的子类型[RFC4519]。

## 2.6\. Alias Entries 别名条目

别名是一个对象或对象条目的另一个名字。条目的别名由别名条目提供。

每一个别名条目都要包含`aliasedObjectName`属性。系统通过这个属性将别名转化为对象原来的名字。这个操作可用户别名条目的检查。

在目录树中，任何特定的条目都可以有0个或多个别名。一个别名条目克制指向量一个别名条目。

一个别名条目不能有下级，因此别名条目一定是叶子节点。

每一个别名条目都应该属于`alias`对象类。`alias`类继承自`top`和`STRUCTURAL`类型。且必须包含`aliasedObjectName`属性。它的定义如下：

```
    ( 2.5.6.1 NAME 'alias'
      SUP top STRUCTURAL
      MUST aliasedObjectName )
```

别名条目举例：

```
    dn: cn=bar,dc=example,dc=com
    objectClass: top
    objectClass: alias
    objectClass: extensibleObject
    cn: bar
    aliasedObjectName: cn=foo,dc=example,dc=com
```

# 3\. Directory Administrative and Operational Information 目录管理和操作模型

本节讨论X.500管理和操作信息模型。

## 3.1\. Subtrees

在X.501中定义如下：

> 子树是位于一棵树节点(原文是顶点，vertices)下的所有的对象和别名条目的集合。子树的根节点一般是DIT根的子节点。

## 3.2\. Subentries

子条目是一种目录中的特殊条目，被用于保存子树及其细节信息。

## 3.3\. The 'objectClass' attribute

每一个条目有一个`objectClass`属性。其定义如下：

```
    ( 2.5.4.0 NAME 'objectClass'
    EQUALITY objectIdentifierMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.38 )
```

其中，等效匹配规则`objectIdentifierMatch`和对象标识符语法`1.3.6.1.4.1.1466.115.121.1.38`在RFC4517中定义。

`objectClass`属性指定一个条目的对象类型。这个属性可以被客户端修改，但不能被删除。 服务器_应该_约束这种修改，避免条目的基本结果被改变。比如说，不能将`person`改为`country`。

当创建一个条目，或为条目添加一个`objectClass`的值时，该对象类的父类都会被隐性的添加。所以，服务器在处理对象类修改请求时，要进行一些约束。比如，一个条目含有类`x-a`，`x-a`是`x-b`的子类。当客户端要删除`x-b`时，这个请求就是错误的。

## 3.4\. Operational Attributes

用于服务器的管理和操作。有三种操作属性：目录操作属性，DSA共享操作属性，DSA特定操作属性。

目录操作属性用来表示目录信息模型中的操作和管理信息。包括由服务器维护的操作属性(如：`createTimestamp`)和有用户维护的操作属性(如：`dITContentRules`)。

操作属相一般是不可见的。搜索结果也不能返回，除非明确指定。

不是所有的操作属性都可以被用户修改。

条目一般包括下面4个属性：

- creatorsName: 该条目创建人的的DN
- createTimestamp: 条目创建时间
- modifiersName: 条目修改人
- modifyTimestamp: 条目修改时间

# 4\. Directory Schema 目录模式

按照X.501定义:

> 目录模式(The Directory Schema)是一组定义和目录结构约束

> - 防止创建的子条目包含错误的对象类。比如一个国家是一个个人的子项
> - 防止额外的属性添加到不合适的条目中。比如序列号添加到个人条目(为啥不行，身份证啊！)
> - 方式属性值不符合属性类型的语法。比如，应该是字符串类型的属性赋了一个bit类型的值。

正常情况下，目录模式由一下内容组成：

1. 命名表`Name Form`，定义原始的结构对象类的命名关系
2. 目录树结构规则`DIT Structure Rule`，定义条目相对其他条目的命名规则。
3. 目录树内容规则`DIT Content Rule`，定义条目扩展属性
4. 对象类`Object Class`，定义给定类中哪些属性的必选的，哪些属性是可选的。
5. 属性类型`Attribute Type`
6. 匹配规则`Matching Rule`
7. LDAP语法`LDAP Syntax`，定义在LDAP使用的编码。

## 4.1\. Schema Definitions

### 4.1.1\. Object Class Definitions

```
ObjectClassDescription = LPAREN WSP
    numericoid                 ; object identifier
    [ SP "NAME" SP qdescrs ]   ; short names (descriptors)
    [ SP "DESC" SP qdstring ]  ; description
    [ SP "OBSOLETE" ]          ; not active
    [ SP "SUP" SP oids ]       ; superior object classes
    [ SP kind ]                ; kind of class
    [ SP "MUST" SP oids ]      ; attribute types
    [ SP "MAY" SP oids ]       ; attribute types
    extensions WSP RPAREN

kind = "ABSTRACT" / "STRUCTURAL" / "AUXILIARY"
```

### 4.1.2\. Attribute Types

```
AttributeTypeDescription = LPAREN WSP
    numericoid                    ; object identifier
    [ SP "NAME" SP qdescrs ]      ; short names (descriptors)
    [ SP "DESC" SP qdstring ]     ; description
    [ SP "OBSOLETE" ]             ; not active
    [ SP "SUP" SP oid ]           ; supertype
    [ SP "EQUALITY" SP oid ]      ; equality matching rule
    [ SP "ORDERING" SP oid ]      ; ordering matching rule
    [ SP "SUBSTR" SP oid ]        ; substrings matching rule
    [ SP "SYNTAX" SP noidlen ]    ; value syntax
    [ SP "SINGLE-VALUE" ]         ; single-value
    [ SP "COLLECTIVE" ]           ; collective
    [ SP "NO-USER-MODIFICATION" ] ; not user modifiable
    [ SP "USAGE" SP usage ]       ; usage
    extensions WSP RPAREN         ; extensions

usage = "userApplications"     /  ; user
        "directoryOperation"   /  ; directory operational
        "distributedOperation" /  ; DSA-shared operational
        "dSAOperation"            ; DSA-specific operational
```

### 4.1.3\. Matching Rules

```
MatchingRuleDescription = LPAREN WSP
    numericoid                 ; object identifier
    [ SP "NAME" SP qdescrs ]   ; short names (descriptors)
    [ SP "DESC" SP qdstring ]  ; description
    [ SP "OBSOLETE" ]          ; not active
    SP "SYNTAX" SP numericoid  ; assertion syntax
    extensions WSP RPAREN      ; extensions
```

# 5\. DSA (Server) Informational Model 服务器信息模型

LDAP协议假设有一个或多个服务器共同提供目录访问服务。保存原始信息的服务器称为`master`，保存副本信息的服务器称为`shadowing`或 `caching`服务器。

目录树的根是一个特殊的服务器条目，每个服务器都有一个不同的属性值。

## 5.1\. Server-Specific Data Requirements

一个LDAP服务器_应该_提供关于它本身的信息和相对其他服务器不同的特定的信息。这些内容在根DSE(DSA-specific Entry 服务器特有条目，其RND为空)中体现。

服务器允许客户端适当的修改根DSE。

本文定义了一下根DSE属性:

- altServer，备份服务器
- namingContexts，命名上下文
- supportedControl，支持的LDAP控制请求
- supportedExtension，支持的LDAP扩展操作
- supportedFeatures，支持的LDAP特性
- supportedLDAPVersion，支持的LDAP版本
- supportedSASLMechanisms，支持的SASL认证机制

# 6\. Other Considerations 其他

# 7\. Implementation Guidelines 目录服务实现指导原则

## 7.1\. 服务器实现指导原则

服务器**必须**识别所有的，在本文中定义的，属性类型名字和对象类名字。但是，除非另有说明，不需要支持相关功能。服务器_应该_识别章节3和章节4定义的所有属性类型和对象类的名字。

服务器**必须**确保条目符合用户和系统模式规则，或其他数据模型约束

服务器可以支持DIT内容规则。服务器可以支持DIT结构规则和命名方式。

服务器可以支持别名条目。

服务器可以支持子条目。

服务器可以实现额外的模式元素。服务器_应该_提供所有支持模式元素的定义。

## 7.2\. 客户端实现指导原则

如果客户端没有与服务器达成预先的协议，客户端_不应该_嘉定服务器支持任何特殊的，超出[章节7.1](#71-服务器实现指南)提到的模式元素。客户端可以通过[4.4节](#44)描述的方法获取子模式(subschema)信息。

客户端**不能**显示或尝试按照ASN.1解码一个语法未知的值。客户端**不能**假设LDAP指定字符编码是UTF8编码或任何只能的Unicode子集，除非这样的限制明确规定。客户端_不应该_在请求信息语法无效的时候发送属性值数据。
