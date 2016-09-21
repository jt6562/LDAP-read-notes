OpenLDAP2.4管理员指南

[toc]

# 1.OpenLDAP介绍
略。

# 2.快速开始指南
以下是一个OpenLDAP2.4的快速开始指南, 包含独立的LDAP演示, slapd(8)。

它包括安装和配置OpenLDAP软件所需要的基本步骤。但是你应该结合其它文档一起来看，包括本文的其他章节,手册页面,以及其随同软件提供的材料(例如INSTALL文档)或[OpenLDAP网站](http://www.OpenLDAP.org)，特别是[OpenLDAP的常见问题解答](http://www.OpenLDAP.org/faq/?file=2)。

如果你确定要使用OpenLDAP软件, 你应该在尝试安装软件之前阅读本文的全文。

> 注意: 这个快速开始指南没有使用强验证或任何保密服务。这些服务在本OpenLDAP管理员指南的其它章节里有所描述。 

## 1.获得软件

你可以从[OpenLDAP软件下载页面](http://www.openldap.org/software/download/)获得一份软件拷贝。建议新用户使用最新版本。

## 2.解压压缩包

给源码选择一个目录, 进入到那个目录, 使用以下命令解压发行版:
```gunzip -c openldap-VERSION.tgz | tar xvfB -```
然后进入目录：
```cd openldap-VERSION```
你要把VERSION换成相应的版本名。


## 3.阅读文档
你现在应该阅读压缩包所提供的COPYRIGHT, LICENSE, README 和 INSTALL 文档。 COPYRIGHT 和 LICENSE 提供的信息是关于可接受的使用,拷贝方式和OpenLDAP软件的使用限制。

你也应该阅读本文的其他章节。特别是, 本文的编译和安装OpenLDAP软件章节提供了依赖的软件的详细信息和安装过程。

## 4.运行configure
你将需要运行提供的configure脚本来配置OpenLDAP，使它能在你的系统上进行编译。configure脚本接受很多命令行选项以打开或关闭可选的软件功能。通常缺省就能够使用,但你可以改变它们。要获得configure可接受的选项的完整列表, 使用 --help 选项: 
```./configure --help```

无论如何, 是你在使用这个指南, 我们假定你足够勇敢,就让configure决定什么是最好的: 
```
./configure
```

假定configure不喜欢你的系统,你可以继续编译软件。如果configure报错, 那么, 你可能需要去看看[软件常见问题解答的安装一节](http://www.openldap.org/faq/?file=8) 或仔细阅读本文的编译和安装OpenLDAP软件一章。 

## 5.编译软件
下一步是编译软件. 这一步分为两部分,首先我们构建依赖，然后编译软件: 
```
make depend
make
```
两个 make 都应该不出错地完成。

## 6.测试编译结果
为了确保正确的编译,你应该运行测试套件(只要花几分钟): 
```
make test
```
应用你的配置的测试将运行并应该通过. 但是一些测试, 例如复制测试, 可能会跳过。

## 7.安装软件
现在准备开始安装软件;这通常需要超级用户权限: 
```
su root -c 'make install'
```
现在所有文件应该都被安装在/usr/local目录下(或任何configure指定的安装目录下). 

## 8.编辑配置文件
使用你的编辑器编辑附带的slapd.conf(5)例子(通常安装在 /usr/local/etc/openldap/slapd.conf) 来包含一个如下格式的 BDB 数据库定义:
```
database bdb
suffix "dc=<MY-DOMAIN>,dc=<COM>"
rootdn "cn=Manager,dc=<MY-DOMAIN>,dc=<COM>"
rootpw secret
directory /usr/local/var/openldap-data
```
确保以你的域名的适当部分替换`<MY-DOMAIN>`和`<COM>`。例如, 对于 example.com, 使用: 
```
database bdb
suffix "dc=example,dc=com"
rootdn "cn=Manager,dc=example,dc=com"
rootpw secret
directory /usr/local/var/openldap-data
```
如果你的域包含额外的部分, 例如 eng.uni.edu.eu, 使用: 
```
database bdb
suffix "dc=eng,dc=uni,dc=edu,dc=eu"
rootdn "cn=Manager,dc=eng,dc=uni,dc=edu,dc=eu"
rootpw secret
directory /usr/local/var/openldap-data
```
关于配置slapd(8)的细节,可在slapd.conf(5) 手册页，以及本文的 slapd 配置文件一章找到. 注意启动slapd(8)之前那些定义的目录必须实际存在. 

## 9.导入数据库配置
现在准备导入你的数据库配置，运行下面的命令：
```
su root -c /usr/local/sbin/slapadd -F /usr/local/etc/cn=config -l /usr/local/etc/openldap/slapd.ldif
```

## 10.启动slapd
现在你准备启动独立的LDAP守护进程, slapd(8), 运行这个命令: 
```
su root -c /usr/local/libexec/slapd
```
为了检查服务器是否运行以及是否被正确地配置好,你可以使用ldapsearch(1)针对它运行一个搜索。 缺省的, ldapsearch被安装在 /usr/local/bin/ldapsearch: 
```
ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
```
注意在命令参数周围使用单引号来避免shell被特殊字符中断. 它应该返回: 
```
dn:
namingContexts: dc=example,dc=com
```
关于运行slapd(8)的细节可以在slapd(8)手册页以及本文的运行slapd一章找到. 

## 11.添加初始条目到目录中
你可以使用ldapadd(1)添加条目到你的LDAP目录. ldapadd期待的输入是LDIF格式。
我们将分两步走: 
    1. 建立LDIF文件
    2. 运行ldapadd 

使用你的编辑器新建一个LDIF文件，包含如下内容: 
```
dn: dc=<MY-DOMAIN>,dc=<COM>
objectclass: dcObject
objectclass: organization
o: <MY ORGANIZATION>
dc: <MY-DOMAIN>

dn: cn=Manager,dc=<MY-DOMAIN>,dc=<COM>
objectclass: organizationalRole
cn: Manager
```
确保使用你的域名的适当部分替换`<MY-DOMAIN>`和`<COM>`，`<MY ORGANIZATION>`应该被你的机构名称替换掉. 当你剪切粘贴时, 确定本例中的每一行的前面和后面都没有空格. 
```
dn: dc=example,dc=com
objectclass: dcObject
objectclass: organization
o: Example Company
dc: example

dn: cn=Manager,dc=example,dc=com
objectclass: organizationalRole
cn: Manager
```
现在, 你可以运行ldapadd(1)来添加这些条目到你的目录. 
```
ldapadd -x -D "cn=Manager,dc=<MY-DOMAIN>,dc=<COM>" -W -f example.ldif
```
确保用你的域名的适当部分替换`<MY-DOMAIN>`和`<COM>`。你将收到提示输入密码，也就是在slapd.conf中定义的"secret"。例如, 对于 example.com, 使用: 
```
ldapadd -x -D "cn=Manager,dc=example,dc=com" -W -f example.ldif
```
这里example.ldif就是你上面新建的文件。

另外关于建立目录的信息可以在本文的数据库建立和维护工具一章找到。

## 12.检测添加结果
现在我们准备检验目录中添加的条目。你可使用任何LDAP客户端来做这件事,但我们的例子使用ldapsearch(1)工具。记住把 dc=example,dc=com 替换成你的网站的正确的值: 
```
ldapsearch -x -b 'dc=example,dc=com' '(objectclass=*)'
```
本命令将搜索和接收这个数据库中的每一个条目。

现在你准备使用ldapadd(1)或其它LDAP客户端添加更多的条目, 试验更多的配置选项, 后端安排, 等等.

注意缺省的情况下, slapd(8)数据库赋予阅读权限给每个人，除了超级用户(即配置文件中的rootdn参数). 强烈建议你建立控制来限制授权用户的操作。操作权限控制在访问控制章讨论。也鼓励你阅读安全事项，使用SASL和使用TLS章节.

接下来的章节提供更多编译,安装和运行slapd(8)的详细信息。

# 3.大图片-配置选择
本节概述了各种LDAP目录配置，以及如何使您的独立的LDAP守护进程slapd（8）适合世界其他国家。

## 1.本地目录服务
在这种配置中，你运行slapd（8）实例，只为你的本地网域提供目录服务。它不以任何方式与其他目录服务器交互。这种配置如图3.1。

![图3.1 本地服务配置](http://www.openldap.org/doc/admin24/config_local.png)

图3.1 本地服务配置

如果你刚刚起步（快速启动指南的目的之一就是为了你这种人），或者如果你想提供本地服务且对连接到世界其他地区不感兴趣,请使用此配置。如果以后你想的话,这也很容易升级到另一个配置。

## 2.带转发的本地服务
在这种配置中，你运行slapd （ 8 ）实例，为你的本地网域提供目录服务，并配置它返回转发到其他能够处理请求的服务器。你可以自己运行此服务（或多个服务）或使用别人提供给你的服务。这种配置如图3.2。

![图3.2 带转发的本地服务](http://www.openldap.org/doc/admin24/config_ref.png)

图3.2 带转发的本地服务

如果你想提供本地服务，并参与全球目录，或者您想代表负责下属条目到另一台服务器,使用此配置。 

## 3.可复制的目录服务
slapd （ 8 ）包括了对LDAP基于同步的复制的支持， 即所谓syncrepl，可用于在多个目录服务器上维持目录信息的影子复制。在其最基本的配置，主服务器是一个syncrepl供应商，而一个或多个从服务器（或影子服务器）是syncrepl消费者。一个主从配置的例子如图3.3。多主机的配置，也支持。

![图3.3 可复制的目录服务](http://www.openldap.org/doc/admin24/config_repl.png)

图3.3 可复制的目录服务

此配置可用于头两个配置的任何一个情况下，例如一个单一的slapd（8）没有提供所需的可靠性和可用性。

## 4.分布式本地目录服务
在这种配置中，当地的服务被分割成较小的服务，每个都是可复制的，和上下级粘在一起转发。

# 4.编译和安装OpenLDAP软件
这一章详细说明如何编译和安装包含了slapd（8），独立的LDAP守护进程的OpenLDAP软件包。编译和安装OpenLDAP软件需要几个步骤：安装依赖的软件，配置OpenLDAP软件本身，编译，并最终安装。以下各节详细描述了此过程中。 

## 1.获得和解包软件
你可以从该项目的[下载页面上](http://www.openldap.org/software/download/)或直接从该项目的[FTP服务](ftp://ftp.openldap.org/pub/OpenLDAP/)获取OpenLDAP软件。

该项目提供两个系列的包作为一般用途。该项目发布版本提供新的特性和错误修正。虽然该项目采取措施，以改善这些版本的稳定性，但是经常会版本发布之后才发现问题。稳定版本是最新的经过一般性的使用已经显示出稳定的版本。

OpenLDAP软件的用户可以选择最适当的一系列安装，这取决于他们对于最新功能与稳定表现的期望。

OpenLDAP软件下载后，你需要从压缩存档文件提取发布版并更改您的工作目录到发布版的根目录：
```
gunzip -c openldap-VERSION.tgz | tar xf -
cd openldap-VERSION
```
你需要把 VERSION 换成发布版的实际版本号.

你现在应该阅读版权，许可证，自述文件和发布版提供的安装文件规定。版权和许可提供对OpenLDAP软件的可接受的使用，复制，和限制的保证。自述文件和安装文件提供依赖的软件和安装过程的详细资料。

## 2.依赖的软件
OpenLDAP软件依靠一些第三方分发的软件包。根据您打算使用的不同的功能，您可能必须下载并安装一些额外的软件包。本节详述通常需要的您可能需要安装的第三方软件的软件包。然而，一个最新的依赖软件信息，应在自述文件中获得。请注意，其中一些第三方软件包可能依赖于额外的软件包。每个软件包提供了它自己的安装说明。

### 1.传输层安全
OpenLDAP客户端和服务器需要安装OpenSSL或GnuTLS的TLS库来提供TLS，传输层安全服务。虽然一些操作系统可能提供这些库的一部分，作为基本系统或一个可选的软件组件，OpenSSL和GnuTLS往往需要单独安装。

OpenSSL可从[这](http://www.openssl.org/)获得。GnuTLS可从[这](http://www.gnu.org/software/gnutls/)获得.

OpenLDAP 软件将不是完全兼容 LDAPv3,除非 OpenLDAP 的配置检测到一个可用的 TLS 库。

### 2.简单验证和安全层
OpenLDAP客户端和服务器需要安装Cyrus SASL 库提供简单身份认证和安全层服务。虽然一些操作系统可能会提供这个库，作为基本系统的一部分或作为一个可选的软件组件，Cyrus SASL 往往需要单独安装。

Cyrus SASL 可从[这](http://asg.web.cmu.edu/sasl/sasl-library.html)获得. Cyrus SASL 将使用 OpenSSL 和 Kerberos/GSSAPI 库，如果预先安装了的话。

OpenLDAP软件将不完全兼容 LDAPv3，除非 OpenLDAP 的配置检测到一个可用的 Cyrus SASL 安装。

### 3.Kerberos验证服务
OpenLDAP客户端和服务器支持Kerberos身份验证服务。特别是， OpenLDAP支持 Kerberos V GSS-API SASL 认证机制，称为GSSAPI机制。此功能要求，除了Cyrus SASL 库之外，还要有 Heimdal 或 MIT Kerberos V 库。

Heimdal Kerberos 可从[这](http://www.pdc.kth.se/heimdal/)获得. MIT Kerberos 可从 [这](http://web.mit.edu/kerberos/www/)获得。

强烈推荐使用强验证服务, 例如 Kerberos 提供的那些。

### 4.数据库软件
OpenLDAP的slapd （ 8 ）的BDB和HDB主要数据库后端需要甲骨文公司的Berkeley DB。如果在设定的时间没有可用的，您将无法与这些主要的数据库后端编译slapd （ 8 ）。

您的操作系统可能提供一个支持版本的Berkeley DB作为基础系统或作为一个可选的软件组件。如果不是这样，您将不得不自己去获取并安装它。

Berkeley DB 可从 [Oracle 公司的 Berkeley DB 下载页](http://www.oracle.com/technology/software/products/berkeley-db/index.html)获得.

有很多可用的版本. 通常, 推荐最近的版本 (包含发布的补丁) . 如果你想使用BDB或HDB数据库后端，这个包是必需的.

> 注意: 请看推荐的 OpenLDAP 软件依赖版本 一节获得更多信息。

### 5.线程
OpenLDAP设计成充分利用线程。 OpenLDAP支持POSIX pthreads，Mach CThreads ，以及其他一些品种。如果不能找到一个合适的线程子系统,configure会抱怨。如果发生这种情况，请咨询[OpenLDAP常见问题](http://www.openldap.org/faq/)中的 软件|安装|平台 提示部分。

### 6.TCP包装
slapd(8) 支持 TCP 包装 (IP 级访问控制过滤), 如果预先安装了的话. 建议为包含非公开信息的服务器使用 TCP 包装或其他 IP级的访问过滤 (例如那些IP级防火墙所提供的)。

## 3.运行configure
现在，您或许应该运行configure脚本的--help选项。这将给你一个选项列表，编译OpenLDAP时您可以变更这些选项 。使用此方法可以启用或禁用OpenLDAP的许多功能。
```
./configure --help
```
configure脚本还将使用各种环境变量的某些设置。这些环境变量包括：

表 4.1: 环境标量

| 变量        | 描述                    |
| :---------- | :---------------------- |
| CC          | 指定替代的 C 编译器     |
| CFLAGS      | 指定额外的编译器 flags  |
| CPPFLAGS    | 指定 C 预编译器 flags   |
| LDFLAGS     | 指定 linker flags       |
| LIBS        | 指定额外的库            |

现在以任何期望的配置选项或环境变量运行configure脚本.
```
./configure [options] [variable=value ...]
```
作为一个例子，假设我们要安装OpenLDAP,后端是BDB并支持TCP封装。默认情况下，BDB是启用的而TCP封装并非如此。所以，我们只需要指定 --enable-wrappers， 来包含对TCP封装的支持：
```
./configure --enable-wrappers
```
无论如何，这无法定位没有安装到系统目录的依赖的软件. 例如, 如果 TCP Wrappers 头文件和库文件分别被安装在 /usr/local/include 和 /usr/local/lib, configure 脚本应该如下调用:
```
./configure --enable-wrappers \
        CPPFLAGS="-I/usr/local/include" \
        LDFLAGS="-L/usr/local/lib -Wl,-rpath,/usr/local/lib"
```

> 注意: 一些 shells, 例如那些衍生自Bourne sh(1)的 shell, 不需要使用 env(1) 命令. 在某些情况下, 环境变量不得不使用替代的语法来指定。

configure脚本通常将自动检测适当的设定。如果你对此阶段有任何问题,咨询任何平台特定的提示并检查你的configure选项, 如果有的话。

## 4.编译软件
一旦你运行了 configure 脚本,输出的最后一行应该是:
```Please "make depend" to build dependencies```
如果最后一行输出不符, configure失败了,你需要阅读它的输出来决定什么地方出错了。你不应继续下去,直到configure成功的完成。

编译依赖, 运行:
```make depend```
现在编译软件, 这一步将确实地编译OpenLDAP。
```
make
```

你应该仔细检查这个命令的输出信息以确认每件东西正确地被编译了. 注意这个命令同时编译了 LDAP 库文件和相关的客户端以及 slapd(8).

## 5.测试软件
一旦软件被正确地配置和成功地编译了, 你应该运行测试套件来检查这个版本.
```
make test
```
适用于您的配置地试验将运行，它们应该通过。一些测试，如复制测试，可以跳过，如果您的配置不支持的话。 

## 6.安装软件
一旦你成功地测试了软件,你开始准备安装它.你将需要在运行configure时你指定的那些安装目录的写权限. 缺省OpenLDAP软件安装在/usr/local. 如果你用 --prefix configure 选项改变了设置 , 它将被安装在你提出的位置.

典型的, 安装需要超级用户权限. 从OpenLDAP源码根目录, 键入:
```
su root -c 'make install'
```

需要的时候输入适当的密码.

你应该仔细检查此命令的输出以确保每件东西正确地安装了. 缺省你将在 /usr/local/etc/openldap 发现slapd(8)的配置文件. 更多信息见 配置 slapd 一章。

# 5.配置slapd
 一旦该软件已编译并安装后，您准备配置slapd（8），用于您的网站。与以前OpenLDAP的版本不同，slapd （8）运行时配置在2.3（及更高版本）是完全的允许LDAP并且可以LDIF数据使用标准的LDAP操作来管理。LDAP配置引擎允许所有slapd的配置选项在运行中改变，一般不需要重新启动服务器以使更改生效。旧风格slapd.conf（5）文件仍然是支持的，但必须转换为新的slapd配置（5）格式，允许运行时改变被保存。虽然旧的风格配置使用一个单一的文件，通常安装在/usr/local/etc/openldap/slapd.conf，新的风格采用了slapd后端数据库来存储配置。配置数据库通常放在/usr/local/etc/openldap/slapd.d目录。从slapd.conf格式转换成slapd.d格式时，任何包含文件也将被集成到由此产生的配置数据库。

可以通过slapd （ 8 ）的命令行选项指定候补的配置目录（或文件）。本章说明配置系统的一般格式，然后详细说明了常用的配置设置。

> 注：一些后端和分布式覆盖还不支持运行时配置。在这种情况下，旧式slapd.conf （ 5 ）文件必须使用。 

## 1.配置布局
slapd配置作为一个特殊的拥有预定义的规划和DIT的LDAP目录来存储。有一些特定的objectClasses用于进行全球配置选项，架构定义，后端和数据库的定义，以及各种其他项目。样本配置树如图5.1。

![图5.1 样本配置树](http://www.openldap.org/doc/admin24/config_dit.png)

图5.1 样本配置树

其他对象可能是配置的一部分，但省略了明确的说明.

该slapd配置配置树有一个非常特殊的结构。树的根被命名为cn=config，并且包含全局配置设置。其他设置均包含在独立的子条目中：
> * 动态装载模块
这些可能只在使用了 --enable-modules 选项 configure 软件的时候需要用。
> * 规划定义 
cn=schema,cn=config 条目包含了系统的规划(在slapd中所有规划都是硬编码)。
cn=schema,cn=config 的子条目包含从配置文件装载或运行时添加的的用户规划。
> * 特定后端配置
> * 特定数据库配置 
Overlay被定义在数据库条目的子条目下。
数据库和Overlay也可以有其他的杂项子条目。

LDIF文件的通常规则适用于配置信息：以'＃'字符开始的注释行会被忽略。如果一行的开始是一个空格，它被认为是延续前行（即使前行是注释）并且这个单个的空格会被删除。条目是由空白行分开的。

config LDIF的总体布局如下： 
```
# 全局配置设定
dn: cn=config
objectClass: olcGlobal
cn: config
<global config settings>

# 规划定义
dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema
<system schema>

dn: cn={X}core,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: {X}core
<core schema>

# 额外的用户定义的规划
...

# 后端定义
dn: olcBackend=<typeA>,cn=config
objectClass: olcBackendConfig
olcBackend: <typeA>
<backend-specific settings>

# 数据库定义
dn: olcDatabase={X}<typeA>,cn=config
objectClass: olcDatabaseConfig
olcDatabase: {X}<typeA>
<database-specific settings>

# 随后的定义和设置
...
```
一些以上所列项目在他们的名字有一个数字索引"{X}"。虽然大多数配置设置具有内在的次序依赖（即一个设置必须在另一个设定之前生效），LDAP数据库本身是无次序的。这个数字索引是用来在配置数据库强制一致性的次序，以便使所有的次序依赖被保存。在大多数情况下，并没有提供索引，它将根据条目创建的顺序自动生成。

配置指令被定义为一个独立属性的值。大部分用于slapd配置的属性和objectClasses在他们的名字有一个“olc”前缀（OpenLDAP配置）。通常在属性和旧式slapd.conf配置关键字之间有一对一的对应关系，使用关键字作为属性名称，以“olc”作前缀。

一个配置指令可采用参数。如果是这样，参数们由空格分开。如果参数包含空格，这个参数应包含在双引号里“像这样”。在以下描述中，在方括号<>中的参数应被实际的文字替换 。

发布版包含一个配置文件例子，将被安装在/usr/local/etc/OpenLDAP目录。在/usr/local/etc/openldap/schema目录还提供了一些包含架构定义（属性类型和对象类）的文件。

## 2.配置指令
本节详述常用的配置指令。如需完整清单，请参阅slapd-config（5）帮助页面。本节将按自上而下的顺序排列配置指令，首先是全球性指令的`cn=config`项。每个指令将描述其默认值（如果有的话）和它的一个使用例子。
### 1.**`cn=config`**

此项目中所载的指令一般适用于把服务器作为一个整体。他们中的大多数是系统或面向连接的，而不是数据库相关的。此项目必须有olcGlobal对象。
#### 1.`olcIdleTimeout: <integer>`

指定强行关闭闲置客户端连接等待的秒数。默认情况下，值为0，禁用此功能。
#### 2.`olcLogLevel: <level>`

该指令规定在哪一级的调试和运行统计报表应syslogged（目前记录到syslogd （ 8 ） LOG_LOCAL4设施）。您必须配置OpenLDAP -enable-debug（默认）为工作（除了这两个统计级别，是始终启用的）。日志级别可能会被指定为整数或关键字。可使用多重记录级别并且级别是可添加的。要显示哪些层次对应于什么样的调试，以 -D? 调用 slapd 或参考下表。`<level>`可能的值有： 

表5.1 调试级别

| 级别  | 关键字        |  描述                                |
| :---- | :----         | :----                                |
| 0     |               | 不调试                               |
| 1     | (0x1 trace)   | 跟踪函数调用                         |
| 2     | (0x2 packets) | 调试包的处理                         |
| 4     | (0x4 args)    | 重度跟踪调试                         |
| 8     | (0x8 conns)   | 连接管理                             |
| 16    | (0x10 BER)    | 打印发送和接收的包                   |
| 32    | (0x20 filter) | 搜索过滤器的处理                     |
| 64    | (0x40 config) | 配置处理                             |
| 128   | (0x80 ACL)    | 访问控制列表处理                     |
| 256   | (0x100 stats) | 连接/操作/结果的统计日志             |
| 512   | (0x200 stats2)| 发送的条目的统计日志                 |
| 1024  | (0x400 shell) | 打印和shell后端的通信                |
| 2048  | (0x800 parse) | 打印条目解析调试                     |
| 16384 | (0x4000 sync) | syncrepl消费者处理                   |
| 32768 | (0x8000 none) | 只显示那些不受日志级别设置影响的消息 |

预期的日志级别可作为一个单一的整数输入，它结合了 (或运算) 预期级别(们), 包括十进制或十六进制符号, 作为一些整数的列表 (内部或运算), 或作为名字的列表，展示在括号里, 例如 
```
olcLogLevel 129
olcLogLevel 0x81
olcLogLevel 128 1
olcLogLevel 0x80 0x1
olcLogLevel acl trace
```
是等效的。
例子：
```
olcLogLevel -1
```
这将导致很多很多的调试信息将被记录。
```
olcLogLevel conns filter
```
只需登录连接和搜索筛选器处理。
```
olcLogLevel none
```
这将忽略配置文件中日志等级的配置。当没有记录发生，它与将日志级别设置为0是不同的。至少None级别需要记录具有高优先级消息。

默认的配置：
```
olcLogLevel stats
```
日志记录会使用默认配置。然而，如果没有olcLogLevel被定义，没有记录发生（相当于设置为 Level 0）。
#### 3.`olcReferral <URI>`
该指令规定了slapd找不到一个本地的数据库时，跳转到一个指定URI来处理请求。
例如：
```
olcReferral: ldap://root.openldap.org
```
这将指定非本地查询OpenLDAP项目的全局根LDAP服务器。在该服务器上的智能LDAP客户端会重新发起他们的查询，但是请注意，大多数客户只是要知道如何处理包含一个主机部分和可选的DN部分的简单的LDAP的URL。

#### 4.示例条目
```
dn: cn=config
objectClass: olcGlobal
cn: config
olcIdleTimeout: 30
olcLogLevel: Stats
olcReferral: ldap://root.openldap.org
```

### 2.**`cn=module`**
如果在配置的slapd时被启用动态加载模块的支持，可以使用CN=模块条目指定集合的模块加载。模块条目必须有olcModuleLoad对象类。

#### 1.`olcModuleLoad: <filename>`
指定一个可动态加载的模块名称。文件名可以是绝对路径名或一个简单的文件名。非绝对路径名称，会从olcModulePath指令指定的目录中搜索。

#### 2.`olcModulePath: <pathspec>`
指定要搜索加载模块的目录列表。通常，所述路径是冒号分隔但这取决于操作系统。

#### 3.示例条目
```
dn: cn=module{0},cn=config
objectClass: olcModuleList
cn: module{0}
olcModuleLoad: /usr/local/lib/smbk5pwd.la

dn: cn=module{1},cn=config
objectClass: olcModuleList
cn: module{1}
olcModulePath: /usr/local/lib:/usr/local/lib/slapd
olcModuleLoad: accesslog.la
olcModuleLoad: pcache.la
```
### 3.**`cn=schema`** 
在cn=schema条目保存了slapd中所有的硬编码的模式定义。此项目中的值都是由slapd所产生，而不需要在配置文件中提供。但是条目仍然要被定义，以作为用户定义的模式。模式条目必须有olcSchemaConfig对象类。

#### 1.`olcAttributeTypes: <RFC4512 Attribute Type Description>`
该指令定义的属性类型。请参阅有关如何使用该指令的信息模式规范章节。

#### 2.`olcObjectClasses: <RFC4512 Object Class Description>`
该指令定义了一个对象类。请参阅有关如何使用该指令的信息模式规范章节。

#### 3.示例条目
```
dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema

dn: cn=test,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: test
olcAttributeTypes: ( 1.1.1
  NAME 'testAttr'
  EQUALITY integerMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 )
olcAttributeTypes: ( 1.1.2 NAME 'testTwo' EQUALITY caseIgnoreMatch
  SUBSTR caseIgnoreSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.44 )
olcObjectClasses: ( 1.1.3 NAME 'testObject'
  MAY ( testAttr $ testTwo ) AUXILIARY )
```
### 4.具体后端指令
后端指令应用于同一类型的所有数据库实例，并根据该指令，可以通过数据库指令覆盖。后端条目必须有olcBackendConfig对象类。

#### 1.`olcBackend: <type>`
该指令名是后端指令的配置条目。`<type>`应是表5.2中所列的支持后端类型之一。

表5.2 数据库后端

| 类型    | 描述                            |
| :----   | :----                           |
| bdb     | Berkeley DB的事务性后端         |
| config  | slapd配置后端                   |
| dnssrv  | DNS SRV后端                     |
| hdb     | BDB后端的层次变种               |
| ldap    | 轻量级目录访问协议（代理）后端  |
| ldif    | 轻量级的数据交换格式后端        |
| meta    | META目录后端                    |
| monitor | Monito后端                      |
| passwd  | 提供只读访问的passwd（5）       |
| perl    | Perl程序后端                    |
| shell   | shell（extern程序）后端         |
| sql     | SQL程序后端                     |

例如：
```
olcBackend: bdb
```
该条目没有定义其他指令。具体的后端类型可以为他们的特定用途定义附加属性，但至今都未曾经被定义。这样，这些指令通常不会出现在任何实际的配置中。

#### 2.示例条目
```
dn: olcBackend=bdb,cn=config
objectClass: olcBackendConfig
olcBackend: bdb
```

### 5.具体数据库指令
本节的指令被每一种数据库支持。数据库条目必须有olcDatabaseConfig对象类。

#### 1.`olcDatabase: [{<index>}]<type>`
该指令指定特定数据库实例。数字`{<index>}`可以被提供来区分相同类型的多个数据库。一般的索引可以被省略，并且slapd会自动生成它。`<type>`应在表5.2或前端类型列出的支持的后端类型之一。

前端是用于保存应适用于所有的其它数据库的数据库级别选项的一个特殊的数据库。随后的数据库定义也可以覆盖一些前端设置。

配置数据库也是特别的;两者的配置和前端数据库总是隐式地创建，即使它们没有明确地配置，以及它们的任何其他数据库之前被创建。

例如：
```
olcDatabase: bdb
```
这标志着新的BDB数据库实例的开始。

#### 2.`olcAccess: to <what> [ by <who> [<accesslevel>] [<control>] ]+`
该指令授权访问（通过`<accesslevel>`指定）到一组条目和/或属性（由`<what>`指定）由一个或多个请求者（由<who>指定）。请参阅本指南的基本用法的访问控制部分。

> 注意：如果没有olcAccess指令指定，默认的访问控制策略，允许所有用户（包括认证和匿名）读取权限。
> 注：在前端定义的访问控制是附加到所有其他数据库的控制。

#### 3.`olcReadonly { TRUE | FALSE }`
这个指令将数据库设置为“只读”模式。修改数据库中的任何企图将返回“不愿意执行”的错误。

默认：
```
olcReadonly: FALSE
```

#### 4.`olcRootDN: <DN>`
该指令规定了DN可以操作此数据库，而不受访问控制限制或管理限制。DN不必参考在此数据库或在目录中的条目。DN可以指向一个SASL标识。
基于条目的示例：
```
olcRootDN: "cn=Manager,dc=example,dc=com"
```
基于SASL的示例：
```
 olcRootDN: "uid=root,cn=example.com,cn=digest-md5,cn=auth"
```
请参阅本文有关SASL认证身份信息SASL验证部分。

#### 5.`olcRootPW: <password>`
该指令可以用于为rootdn的DN（当的rootdn在数据库内设置一个DN）指定一个密码。
例如：
```
olcRootPW: secret
```
它也允许在RFC2307的形式提供密码的哈希值。slappasswd（8）可以被用于生成密码散列。
例如：
```
olcRootPW: {SSHA}ZKKuqbEKJfKSXhUbHG3fG8MDn9j1v4QN
```
该hash是使用命令`slappasswd -s secret` 生成的。

#### 6.`olcSizeLimit: <integer>`
该指令指定搜索返回的条目的最大数量。
默认：
```
olcSizeLimit: 500
```
请参阅本指南和slapd-config（5）了解更多详情的限制部分。

#### 7.`olcSuffix: <dn suffix>`
该指令指定了将传递给后端数据库的请求的DN的后缀。多个后缀线可以给定，且通常至少有一个必须为每个数据库定义。（有些后端类型，例如前端和监控使用硬编码的后缀，这可能不会在配置中覆盖。）

例如：
```
olcSuffix: "dc=example,dc=com"
```
查询以“dc=example,dc=com”结尾的DN将被传递给后端。

> 注意：当后端接收到一个被选中的查询时，slapd会顺序的在被配置的每一个数据库寻找该后缀值。因此，如果一个数据库后缀是另一个的前缀，它必须在该配置的后面出现。

#### 8.`olcSyncrepl`
```
olcSyncrepl: rid=<replica ID>
    provider=ldap[s]://<hostname>[:port]
    [type=refreshOnly|refreshAndPersist]
    [interval=dd:hh:mm:ss]
    [retry=[<retry interval> <# of retries>]+]
    searchbase=<base DN>
    [filter=<filter str>]
    [scope=sub|one|base]
    [attrs=<attr list>]
    [attrsonly]
    [sizelimit=<limit>]
    [timelimit=<limit>]
    [schemachecking=on|off]
    [bindmethod=simple|sasl]
    [binddn=<DN>]
    [saslmech=<mech>]
    [authcid=<identity>]
    [authzid=<identity>]
    [credentials=<passwd>]
    [realm=<realm>]
    [secprops=<properties>]
    [starttls=yes|critical]
    [tls_cert=<file>]
    [tls_key=<file>]
    [tls_cacert=<file>]
    [tls_cacertdir=<path>]
    [tls_reqcert=never|allow|try|demand]
    [tls_ciphersuite=<ciphers>]
    [tls_crlcheck=none|peer|all]
    [logbase=<base DN>]
    [logfilter=<filter str>]
    [syncdata=default|accesslog|changelog]
```
该指令通过建立当前的slapd（8）运行的syncrepl复制引擎复制消费者网站指定当前数据库作为主内容的副本。主数据库位于由提供参数指定的复制提供站点。副本数据库随时保持最新与使用LDAP内容同步协议主内容。有关该协议的更多信息，请参见RFC4533。

在摆脱参数用于复制消费者服务器内的当前的syncrepl指令，其中`<replica ID>`唯一地标识由当前的syncrepl指令所述的syncrepl规范的鉴定。 `<replica ID>`是非负的，是长度不超过三个十进制的数字。

provider参数指定包含主内容作为一个LDAP URI复制提供者站点。提供者参数指定的方案中，主机和任选其中所述提供商的slapd实例可以发现一个端口。可用于`<hostname>`是一个域名或IP地址。例如LDAP：//provider.example.com：389或LDAPS：//192.168.1.1：636。如果`<port>`没有给出，则使用标准的LDAP端口号（389或636）。注意syncrepl使用消费者发起协议，因此它的规范位于消费者的网站，而复制品规范位于提供商的网站。的syncrepl和replica指令定义了两个独立的复制机制。它们并不代表彼此的复制体。

所述的syncrepl副本的内容是使用作为其结果集的搜索规范中定义。消费者的slapd将根据搜索规范发送搜索请求给提供的slapd。搜索规范包括searchbase，范围，过滤器，ATTRS，attrsonly，的sizeLimit和时限参数，在正常的搜索规范。该searchbase参数没有默认值，必须指定。范围默认为子，过滤器默认为（objectclass=*），ATTRS默认为“*，+”复制所有的用户和业务属性，attrsonly默认情况下未设置。双方的sizeLimit和时限默认为“无限制”，只有正整数或“无限制”可能被指定。

LDAP内容同步协议有两种操作类型：refreshOnly和refreshAndPersist。操作类型由类型参数指定。在refreshOnly操作，下一个同步搜索操作周期的间隔时间改期每次同步操作结束后。间隔由间隔参数指定。它被设置为一天默认。在refreshAndPersist操作，同步搜索仍然是供应商的slapd实例持久。进一步更新主副本将产生searchResultEntry消费者的slapd以持续同步搜索的搜索响应。

如果在复制过程中发生错误时，消费者将尝试根据重试参数这是`<retry interval>`和`<# of retries>`对的列表重新连接。例如，重试=“60 10 300 3”让消费者重试每隔60秒的第10次，然后重试每300秒重试停先下一三次。`+`在`<# of retries>`中指重试，直到成功的数目不定。

模式检查可以在LDAP同步消费者现场通过接通schemachecking参数来执行。如果它被接通时，每个复制条目将被检查其模式作为条目存储到副本的内容。在副本的每个条目应该包含的架构定义所需的属性。如果它被关闭，参赛作品将被保存而不检查模式的一致性。默认是关闭的。

在指定binddn参数给出了DN绑定作为搜索的syncrepl的提供者的slapd。它应是有读权限在master数据库中复制内容的DN。

该bindmethod是简单或SASL，取决于简单的基于密码的认证或SASL认证是否是连接时的提供者的slapd实例中使用。

简单认证不应该使用，除非有充分的数据完整性和机密性的保护措施（如TLS或IPsec）。简单认证需要指定binddn和credentials参数。

一般推荐SASL认证。 SASL认证需要使用saslmech参数的机制规范。根据不同的机制，认证身份和/或可使用authcid和证书，分别被指定的凭据。所述authzid参数可以被用来指定一个授权的身份。

realm参数指定了一定的验证机制中的身份的境界。该secprops参数指定赛勒斯SASL安全属性。

启动TLS参数指定使用扩展操作认证的供应商之前建立一个TLS会话启动TLS的。如果关键参数提供，如果启动TLS请求失败的会话将被中止。否则的syncrepl会话继续不使用TLS。该TLS_REQCERT设置默认为“需求”和其他TLS设置默认为相同的主slapd的TLS设置。

而不是复制整个项目，消费者可查询的数据修改日志。这种操作模式被称为增量的syncrepl。除了上述参数，logbase和logfilter参数必须被适当地为将要使用的日志设定。所述syncdata参数必须被设置为“ACCESSLOG”如果日志符合slapo-ACCESSLOG（5）日志格式，或“更改日志”如果日志符合过时更改日志格式。如果省略或设置为“默认”的syncdata参数则日志参数将被忽略。

该的syncrepl复制机制是由BDB和HDB后端支持。

请参见本指南的LDAP同步复制一章关于如何使用此指令的更多信息。

#### 9.`olcTimeLimit: <integer>`
该指令指定slapd回答一个搜索请求将花费的最大秒数（实时）。如果一个请求不是在这个时间内结束时，将返回超时提示。
默认：
```
olcTimeLimit: 3600
```
请参阅本指南和slapd-config（5）了解更多详情的限制部分。

#### 10.`olcUpdateref: <URL>`
该指令只适用于一个附属slapd。它指定的URL返回给客户端提交其在副本更新请求。如果指定了多次，每个URL都要提供。
例如：
```
olcUpdateref:   ldap://master.example.net
```

#### 11.示例目录
```
dn: olcDatabase=frontend,cn=config
objectClass: olcDatabaseConfig
objectClass: olcFrontendConfig
olcDatabase: frontend
olcReadOnly: FALSE

dn: olcDatabase=config,cn=config
objectClass: olcDatabaseConfig
olcDatabase: config
olcRootDN: cn=Manager,dc=example,dc=com
```

### 6.BDB和HDB数据库指令
这一类指令同时适用于BDB和HDB数据库。它们除了有上面定义的通用数据库的指令外，还有一个olcDatabase条目。对于BDB/HDB配置指令的完整参考，请参阅slapd-bdb（5）。除了olcDatabaseConfig对象类，BDB和HDB数据库条目必须有olcBdbConfig和olcHdbConfig对象类。

#### 1.`olcDbDirectory: <directory>`
该指令指定包含数据库的BDB文件和相关的索引所在的目录。
默认：
```
olcDbDirectory: /usr/local/var/openldap-data
```

#### 2.`olcDbCachesize: <integer>`
该指令规定了由BDB后台数据库实例维护的内存缓存条目数。
默认：
```
olcDbCachesize: 1000
```

#### 3.`olcDbCheckpoint: <kbyte> <min>`
该指令指定BDB事务日志的检查点的时间。在检查点操作刷新数据库缓冲区写入磁盘，并在日志中写入一个检查点记录。如果`<kbyte>`数据已经写入会发生检查点或`<min>`分钟自上次检查点通过。两个参数默认到零，在这种情况下，它们将被忽略。当`<min>`参数不为零，内部任务运行的每个`<min>`分钟执行检查点。详情请参见Berkeley DB的参考指南。
例如：
```
olcDbCheckpoint: 1024 10
```

#### 4.`olcDbConfig: <DB_CONFIG setting>`
该属性指定放置在数据库目录中的DB_CONFIG文件中的配置指令。在服务器启动时，如果没有这样的文件存在然而，DB_CONFIG文件将被创建，并在该属性的设置将被写入其中。如果该文件存在，其内容将被读取并在此属性显示。该属性是多值的，以适应多重配置指令。没有默认设置，但有必要在这里使用适当的设置，以获得最佳的服务器的性能。

这个属性所做的任何更改将被写入到文件DB_CONFIG并会导致数据库环境中重新使更改立即生效。如果环境缓存大且最近尚未检查点，该复位操作可能需要很长的时间。这可能是最好使用LDAP修改修改这个属性之前，需要手动执行使用Berkeley DB的db_checkpoint实用单检查点。
例如：
```
olcDbConfig: set_cachesize 0 10485760 0
olcDbConfig: set_lg_bsize 2097512
olcDbConfig: set_lg_dir /var/tmp/bdb-log
olcDbConfig: set_flags DB_LOG_AUTOREMOVE
```
在这个例子中，BDB缓存设置为10MB，则BDB事务日志缓冲器大小设置为2MB，和事务日志文件将被存储在/ var / TMP / BDB-日志目录。还一个标志被设置为告诉BDB尽快删除事务日志文件作为其内容已被检查点和它们不再需要。如果没有此设置的事务日志文件会不断积累，直到其他清理过程中删除。详情请参阅关于该db_archive命令Berkeley DB的文档。对于Berkeley DB的标志的完整列表，请参阅 - http://www.oracle.com/technology/documentation/berkeley-db/db/api_c/env_set_flags.html

理想的是，BDB缓存必须至少一样大的数据库的工作集，日志缓冲区的大小应该足够大，以适应大多数交易没有溢出，并且日志目录必须从主数据库文件的单独的物理磁盘上。而且两者的数据库目录和日志目录应该从用于常规的系统活动，如根设备，引导或交换文件系​​统的磁盘分开。请参见常见问题解答-O-MATIC及更多细节Berkeley DB的文档。

#### 5.`olcDbNosync: { TRUE | FALSE }`
此选项会导致磁盘上的数据库内容不会立即与时变存储器的变化同步。将此选项设置为TRUE可改善数据完整性的代价性能。该指令与使用下面的指令相同的效果：
```
olcDbConfig: set_flags DB_TXN_NOSYNC
```

#### 6.`olcDbIDLcacheSize: <integer>`
指定内存索引高速缓存的大小，在索引时隙。默认值是零。较大的数值将加快索引条目频繁搜索。最佳尺寸将取决于该数据库的数据和搜索的特性，但使用数三次条目缓存大小是一个很好的起点。
例如：
```
olcDbIDLcacheSize: 3000
```

#### 7.`olcDbIndex: {<attrlist> | default} [pres,eq,approx,sub,none]`
该指令规定了以保持给定的属性的指标。如果只有一个`<attrlist>`给出，默认索引得以保持。索引关键字对应于常见类型的匹配，可能在一个LDAP搜索过滤器使用。
例如：
```
olcDbIndex: default pres,eq
olcDbIndex: uid
olcDbIndex: cn,sn pres,eq,sub
olcDbIndex: objectClass eq
```
第一行设置的默认设置`pres,eq`。第二行设置默认（`pres，eq`）必须保持uid属性类型。第三行设置`pres,eq`，和子串必须保持cn和sn属性类型。第四行设置的objectClass属性类型为`eq`。

有些不平等的匹配中没有索引关键字。通常这些匹配不使用索引。然而，某些属性不支持索引不平等匹配，而支持平等索引。

子串索引可以更明确地指定为subinitial，subany或subfinal，对应一个字符串匹配过滤的三种可能的组成部分。subinitial只索引子串的开始属性值。subfinal索引子串的结尾属性值，subany索引子串的任何地方。

请注意，在默认情况下，设置一个属性也影响该属性的每个子类型的索引。例如，在名称属性设置一个平等索引将导致CN，SN，和每一个从它的名字继承所有其他属性，被索引。

缺省情况下，索引得以维持。人们普遍注意，在极小的objectClass的平等指数进行维护。
```
olcDbindex: objectClass eq
```
附加索引应该对应于在数据库上使用的最常见的搜索进行配置。Presence索引不应被配置为一个属性，除非该属性在数据库中很少发生，并且正常使用的目录中的属性Presence搜索发生非常频繁。大多数应用程序不使用Presence搜索，所以通常Presence索引不是非常有用。

如果当slapd运行时此设置被改变，内部任务将运行产生改变的索引数据。所有服务器的操作都可以继续在索引器上正常工作。如果索引任务完成之前slapd停止，将不得不使用工具slapindex来手动完成索引工作。

#### 8.`olcDbLinearIndex: { TRUE | FALSE }`
如果此设置为TRUE，slapindex将一次索引一个属性。默认设置为FALSE，这种情况下，条目的所有索引的属性同时被处理。为TRUE时，每个索引的属性被单独处理，在整个数据库中使用多遍。当数据库大小超过BDB缓存大小，此选项可提高性能。当BDB缓存足够大，则不需要此选项，否则会降低性能。此外，默认情况，slapadd执行全索引，因此不需要单独运行的slapindex。打开这个选项，slapadd将不会索引，必须使用slapindex命令来索引。

#### 9.`olcDbMode: { <octal> | <symbolic> }`
该指令规定，新创建的数据库索引文件应有的文件保存模式。可能是`0600`或者`rw----`。
默认：
```
olcDbMode: 0600
```

#### 10.`olcDbSearchStack: <integer>`
指定用于搜索过滤器堆栈的深度。搜索过滤器在堆栈上进行评估，以适应嵌套AND / OR子句。一个单独的堆栈分配给每个服务器线程。堆栈的深度决定了复杂的过滤器可以无需任何附加的存储器分配进行评价。这比搜索堆栈深度嵌套更深的过滤器会导致单独的堆栈中分配给特定的搜索操作。这些独立的分配可以对服务器性能产生重大的负面影响，但指定太多堆栈也会消耗的内存很大。每个搜索使用每级512K字节的32位计算机上，或每级1024K字节的64位计算机上。默认堆栈深度为16，从而8MB或每个线程16MB分别用在32位和64位的机器。另外一个栈槽的512KB大小由编译时常如果需要，它可以改变设置;代码必须重新编译以使更改生效。
默认：
```
olcDbSearchStack: 16
```

#### 11.`olcDbShmKey: <integer>`
指定共享内存BDB环境的关键。默认情况下，BDB环境使用内存映射文件。如果指定了一个非零值时，它将被用作密钥以确定将容纳环境的共享存储器区域。
例如：
```
olcDbShmKey: 42
```

#### 12.示例目录
```
dn: olcDatabase=hdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcHdbConfig
olcDatabase: hdb
olcSuffix: "dc=example,dc=com"
olcDbDirectory: /usr/local/var/openldap-data
olcDbCacheSize: 1000
olcDbCheckpoint: 1024 10
olcDbConfig: set_cachesize 0 10485760 0
olcDbConfig: set_lg_bsize 2097152
olcDbConfig: set_lg_dir /var/tmp/bdb-log
olcDbConfig: set_flags DB_LOG_AUTOREMOVE
olcDbIDLcacheSize: 3000
olcDbIndex: objectClass eq
```

## 3.配置文件示例
下面是一个配置示例，说明文字会穿插其中。它定义了两个数据库来处理X.500树的不同部分;两者都是BDB数据库实例。显示行号仅作参考，不包含在实际的文件。首先，全局配置部分：
```
  1.    # example config file - global configuration entry
  2.    dn: cn=config
  3.    objectClass: olcGlobal
  4.    cn: config
  5.    olcReferral: ldap://root.openldap.org
  6.
```
第1行是一条注释。2-4行标识为全局配置条目。olcReferral：在第5行，意思为查询不是本地的定义的数据库，将被称为在主机root.openldap.org的标准端口（389）上运行的LDAP服务器。 第6行是一个空行，表示该条目结尾。
```
  7.    # internal schema
  8.    dn: cn=schema,cn=config
  9.    objectClass: olcSchemaConfig
 10.    cn: schema
 11.
```
第7行是一条注释。 8-10行标识为架构子树的根。在这个条目中的实际模式定义被硬编码到slapd，因此不需要额外的属性在这里指定。第11行是一个空行，表示该条目结尾。
```
 12.    # include the core schema
 13.    include: file:///usr/local/etc/openldap/schema/core.ldif
 14.
```
第12行是一条注释。 第13行是一个LDIF　include指令访问LDIF格式的核心概要定义。第14行是一个空行。

接下来是数据库定义。第一个数据库是特定的前端数据库，其设置全局应用到所有的其他数据库。
```
 15.    # global database parameters
 16.    dn: olcDatabase=frontend,cn=config
 17.    objectClass: olcDatabaseConfig
 18.    olcDatabase: frontend
 19.    olcAccess: to * by * read
 20.
```
第15行是一条注释。16-18行标识该条目为全局数据库条目。19行是一个全局性的访问控制。它适用于所有条目（任何适用的特定数据库的访问控制后）。第20行是一个空行。

接下来的条目定义的配置后端。
```
 21.    # set a rootpw for the config database so we can bind.
 22.    # deny access to everyone else.
 23.    dn: olcDatabase=config,cn=config
 24.    objectClass: olcDatabaseConfig
 25.    olcDatabase: config
 26.    olcRootPW: {SSHA}XKYnrjvGT3wZFQrDD5040US592LxsdLy
 27.    olcAccess: to * by * none
 28.
```
第21-22行是注释。 23-25行标识该条目配置数据库条目。26行该数据库定义了超级用户密码。（DN将默认为“cn=config”。）27行拒绝所有访问该数据库，因此只有超级用户能够访问它。 （这是配置数据库的默认访问。它只是在这里列出的说明，并重申，除非用一种方法，以作为超级用户明确配置认证，配置数据库将无法访问。）

第28行是一个空行。

下一个条目定义了一个BDB后端将处理事情的查询在树的“dc=example,dc=com”部分。索引将被保持几个属性，而userPassword属性是被保护免受未授权的访问。
```
 29.    # BDB definition for example.com
 30.    dn: olcDatabase=bdb,cn=config
 31.    objectClass: olcDatabaseConfig
 32.    objectClass: olcBdbConfig
 33.    olcDatabase: bdb
 34.    olcSuffix: dc=example,dc=com
 35.    olcDbDirectory: /usr/local/var/openldap-data
 36.    olcRootDN: cn=Manager,dc=example,dc=com
 37.    olcRootPW: secret
 38.    olcDbIndex: uid pres,eq
 39.    olcDbIndex: cn,sn pres,eq,approx,sub
 40.    olcDbIndex: objectClass eq
 41.    olcAccess: to attrs=userPassword
 42.      by self write
 43.      by anonymous auth
 44.      by dn.base="cn=Admin,dc=example,dc=com" write
 45.      by * none
 46.    olcAccess: to *
 47.      by self write
 48.      by dn.base="cn=Admin,dc=example,dc=com" write
 49.      by * read
 50.
```
第29行是注释。30-33行标识该条目作为BDB数据库配置条目。34行指定DN后缀查询传递到该数据库。第35行指定了数据库文件将保存的目录。

36和37行标识数据库超级用户输入和相关联的密码。该条目不受访问控制，大小或时间限制的限制。

38至40行表示的索引，以保持各种属性。

41至49行指定在该数据库中的条目的访问控制。对于所有适用的条目，userPassword属性是该项本身以及由“管理员”条目写入。它可以用于认证/授权的目的，但在其他方面无法读取。所有其它属性都是由项和“管理员”项可写，但可能会被所有用户（认证与否）读取。

50行是一个空行，表示该条目结尾。

下一个条目定义了另一个BDB数据库。这一个处理涉及dc=example,dc=net子树查询，但由相同的实体作为第一数据库来管理。请注意，如果没有第60行，读访问将由于在第19行全局访问规则而被允许。
```
 51.    # BDB definition for example.net
 52.    dn: olcDatabase=bdb,cn=config
 53.    objectClass: olcDatabaseConfig
 54.    objectClass: olcBdbConfig
 55.    olcDatabase: bdb
 56.    olcSuffix: "dc=example,dc=net"
 57.    olcDbDirectory: /usr/local/var/openldap-data-net
 58.    olcRootDN: "cn=Manager,dc=example,dc=com"
 59.    olcDbIndex: objectClass eq
 60.    olcAccess: to * by users read
```

## 4.转换旧的slapd.conf（5）文件到cn=config格式
转换到cn=config格式之前，你应该确保配置后端在现有的配置文件中配置正确。虽然配置后端始终存在slapd里面，默认情况下它仅仅是由它的rootDN访问，除非你明确地配置来验证它的手段。没有默认凭据，这将是无法使用的。

如果你没有准备好一个数据库配置部分，添加这下面的部分在slapd.conf的最后
```
 database config
 rootpw VerySecret
```

> 注：由于配置后端可以用于任意代码加载到slapd进程，仔细防范任何用于访问它的凭据是非常重要的。因为简单的密码很容易受到口令猜测攻击，它通常是更好地忽略rootpw和只使用SASL认证为配置的rootdn。

现有的slapd.conf（5）文件可以被转换新格式使用slaptest（8）或任何其他的dlap工具：
```
slaptest -f /usr/local/etc/openldap/slapd.conf -F /usr/local/etc/openldap/slapd.d
```
你可以使用默认的rootDN及上面配置的rootpw来测试cn=config下条目：
```
ldapsearch -x -D cn=config -w VerySecret -b cn=config
```
然后，您可以丢弃旧的slapd.conf（5）文件。确保使用slapd（8）以及-F选项来指定配置目录，如果你不使用默认的目录路径。

> 注意：当从slapd.conf中格式转换为slapd.d格式，任何包含的文件也将被整合到所得配置的数据库中。

# 6.slapd配置文件
本章介绍通过的slapd.conf（5）配置文件配置slapd（8）。slapd.conf（5）已被弃用，如果您的网站后端需求尚未更新，更新slapd-config（5）使系统正常工作。通过在前面的章节中描述的slapd-Config（5）来配置slapd（8）。

slapd.conf（5）文件通常安装在`/usr/local/etc/openldap`目录。备用的配置文件位置可以通过slapd（8）的命令行选项来指定。

## 1.配置文件格式
slapd.conf（5）文件由三种类型的配置信息：全局的，特定后台和特定数据库的。全局信息被首先指定，接着用特定的后端类型，随后是与特定的数据库实例相关联的信息。全局指令可以在后端或数据库的指令覆盖，后端的指令可以通过数据库指令覆盖。

空行和以“＃”字符开始的注释行将被忽略。如果某行以空格开始，它被认为是上一行的延续（即使前行是注释）。

slapd.conf中的一般格式如下：
```
# global configuration directives
<global config directives>

# backend definition
backend <typeA>
<backend-specific directives>

# first database definition & config directives
database <typeA>
<database-specific directives>

# second database definition & config directives
database <typeB>
<database-specific directives>

# second database definition & config directives
database <typeA>
<database-specific directives>

# subsequent backend & database definitions & config directives
...
```
配置指令可能需要多个参数。如果是这样，用空格将他们隔开。如果一个参数包含空格，参数应该用双引号“like this”。如果一个参数包含双引号或反斜杠字符'\'，这个字符前面应该有一个转义字符'\'。

发行版包含一个安装在/usr/local/etc/openldap目录的示例配置文件。在/usr/local/etc/openldap/schema目录还提供了许多包含模式定义（属性类型和对象类）的文件。

## 2.配置文件指令
本节详细介绍了常用的配置指令。完整列表，请参阅slapd.conf（5）手册页。本节将分别介绍配置文件指令，全局的，后端特定的和数据类别的，描述每个指令及其默认值（如果有的话），并给予其使用的一个例子。

### 1.全局指令
在本节中描述的指令应用于所有的后台和数据库，除非在后台或者数据库的定义中被明确覆盖。应该被实际文本替换的参数显示在括号<>中。

#### 1.`access to <what> [ by <who> [<accesslevel>] [<control>] ]+`
该指令授权访问（通过`<accesslevel>`指定）到一组条目或属性（由`<what>`指定）由一个或多个请求者（由`<who>`指定）。请参阅本指南的基本用法的访问控制部分。

> 注：如果没有指定访问指令，默认的访问控制策略，允许所有注册和非匿名用户读取权限。

#### 2.`attributetype <RFC4512 Attribute Type Description>`
该指令定义一个属性类型。请参阅有关如何使用该指令的信息模式规范章节。

#### 3.`idletimeout <integer>`
指定超时时间来强制关闭空闲的客户端连接。默认超时时间为０，禁用此功能。

#### 4.`include <filename>`
该指令指定，slapd应该在读取本文件的下一行之前，从给定文件中读取其他配置信息。所包含的文件应该是正常的slapd配置文件的格式。该文件通常是包含概要规范的文件。

> 注意：你应该小心使用该指令 - 该指令并没有嵌套include指令层数的限制，也没有环路检测。

#### 5.`loglevel <level>`
该指令指定了调试语句和操作统计输出到syslogged的级别（当前日志输出到syslogd（8）的LOG_LOCAL4中）。您必须通过OpenLDAP的--enable-debug（默认值）打开调试模式（除了两个统计级别，它们总是被启用）。日志级别可被指定为整数或关键字。多个日志级别可以使用，并且等级是附加的。要显示什么数字对应什么样的调试，运行slapd　-d？或查询下面的表格。`<integer>`可能的值如下：

| 级别  | 关键字        |  描述                                |
| :---- | :----         | :----                                |
| -1    | any           | 打开所有调试　                       |
| 0     |               | 不调试                               |
| 1     | (0x1 trace)   | 跟踪函数调用                         |
| 2     | (0x2 packets) | 调试包的处理                         |
| 4     | (0x4 args)    | 重度跟踪调试                         |
| 8     | (0x8 conns)   | 连接管理                             |
| 16    | (0x10 BER)    | 打印发送和接收的包                   |
| 32    | (0x20 filter) | 搜索过滤器的处理                     |
| 64    | (0x40 config) | 配置处理                             |
| 128   | (0x80 ACL)    | 访问控制列表处理                     |
| 256   | (0x100 stats) | 连接/操作/结果的统计日志             |
| 512   | (0x200 stats2)| 发送的条目的统计日志                 |
| 1024  | (0x400 shell) | 打印和shell后端的通信                |
| 2048  | (0x800 parse) | 打印条目解析调试                     |
| 16384 | (0x4000 sync) | syncrepl消费者处理                   |
| 32768 | (0x8000 none) | 只显示那些不受日志级别设置影响的消息 |

所需的日志级别可作为一条结合了（或运算）所需的级别的单一整数被输入，无论是十进制或十六进制的整数，作为整数的列表（在内部或运算），或者如上表所显示的列表括号中的名称，如下：
```
loglevel 129
loglevel 0x81
loglevel 128 1
loglevel 0x80 0x1
loglevel acl trace
```
是等价的。
例如：
```
loglevel -1
```
这将导致很多很多的调试信息被记录。
```
loglevel conns filter
```
只需登录连接和搜索筛选器处理。
```
loglevel none
```
这将忽略配置文件中日志等级的配置。当没有记录发生，它与将日志级别设置为0是不同的。至少None级别需要记录具有高优先级消息。

默认的配置：
```
olcLogLevel stats
```
日志记录会使用默认配置。然而，如果没有olcLogLevel被定义，没有记录发生（相当于设置为 Level 0）。

#### 6.`objectclass <RFC4512 Object Class Description>`
该指令定义了一个对象类。请参阅有关如何使用该指令的信息模式规范章节。

#### 7.`referral <URI>`
该指令规定了slapd找不到一个本地的数据库时，跳转到一个指定URI来处理请求。
例如：
```
referral: ldap://root.openldap.org
```
这将指定非本地查询OpenLDAP项目的全局根LDAP服务器。在该服务器上的智能LDAP客户端会重新发起他们的查询，但是请注意，大多数客户只是要知道如何处理包含一个主机部分和可选的DN部分的简单的LDAP的URL。

#### 8.`sizelimit <integer>`
该指令指定搜索返回的条目的最大数量。
默认：
```
sizeLimit: 500
```
请参阅本指南和slapd.conf（5）了解更多详情的限制部分。

#### 9.`timelimit <integer>`
该指令指定slapd回答一个搜索请求将花费的最大秒数（实时）。如果一个请求不是在这个时间内结束时，将返回超时提示。
默认：
```
timeLimit: 3600
```
请参阅本指南和slapd.conf（5）了解更多详情的限制部分。

### 2.通用后端指令
在本节的指令只应用于定义它们后端。它是由每一种类型的后端的支持。后端指令应用于同一类型的所有数据库实例，并且该指令，可以通过数据库指令覆盖。

#### 1.`backend <type>`
该指令名是后端指令的配置条目。`<type>`应是表5.2中所列的支持后端类型之一。

表6.2 数据库后端

| 类型    | 描述                            |
| :----   | :----                           |
| bdb     | Berkeley DB的事务性后端         |
| config  | slapd配置后端                   |
| dnssrv  | DNS SRV后端                     |
| hdb     | BDB后端的层次变种               |
| ldap    | 轻量级目录访问协议（代理）后端  |
| ldif    | 轻量级的数据交换格式后端        |
| meta    | META目录后端                    |
| monitor | Monito后端                      |
| passwd  | 提供只读访问的passwd（5）       |
| perl    | Perl程序后端                    |
| shell   | shell（extern程序）后端         |
| sql     |  SQL程序后端                    |

例如：
```
backend: bdb
```
该条目没有定义其他指令。具体的后端类型可以为他们的特定用途定义附加属性，但至今都未曾经被定义。这样，这些指令通常不会出现在任何实际的配置中。

### 3.通用数据库指令
本节的指令只应用于定义它们的数据库。他们被每一种数据库支持。

#### 1.`database <type>`
该指令标志着数据库实例声明的开始。`<type>`应是表6.2中所列的支持后端类型之一。
例如：
```
database bdb
```
这标志着一个新的BDB数据库实例声明的开始。

#### 2.`limits <who> <limit> [<limit> [...]]`
根据谁发起的操作指定时间和大小限制。

请参阅本指南和slapd.conf（5）了解更多详情的限制部分。

#### 3.`readonly { on | off }`
该指令将数据库设置为“只读”模式。修改数据库中的任何企图将返回“不愿意执行”的错误。

默认：
```
readonly off
```

#### 4.`rootdn <DN>`
该指令规定了DN可以操作此数据库，而不受访问控制限制或管理限制。DN不必参考在此数据库或在目录中的条目。DN可以指向一个SASL标识。
基于条目的示例：
```
rootDN: "cn=Manager,dc=example,dc=com"
```
基于SASL的示例：
```
 rootDN: "uid=root,cn=example.com,cn=digest-md5,cn=auth"
```
请参阅本文有关SASL认证身份信息SASL验证部分。

#### 5.`rootpw: <password>`
该指令可以用于为rootdn的DN（当的rootdn在数据库内设置一个DN）指定一个密码。
例如：
```
rootpw: secret
```
它也允许在RFC2307的形式提供密码的哈希值。slappasswd（8）可以被用于生成密码散列。
例如：
```
rootpw: {SSHA}ZKKuqbEKJfKSXhUbHG3fG8MDn9j1v4QN
```
该hash是使用命令`slappasswd -s secret` 生成的。

#### 6.`suffix: <dn suffix>`
该指令指定了将传递给后端数据库的请求的DN的后缀。多个后缀线可以给定，且通常至少有一个必须为每个数据库定义。

例如：
```
suffix: "dc=example,dc=com"
```
查询以“dc=example,dc=com”结尾的DN将被传递给后端。

> 注意：当后端接收到一个被选中的查询时，slapd会顺序的在被配置的每一个数据库寻找该后缀值。因此，如果一个数据库后缀是另一个的前缀，它必须在该配置的后面出现。

#### 7.`syncrepl`
```
olcSyncrepl: rid=<replica ID>
    provider=ldap[s]://<hostname>[:port]
    [type=refreshOnly|refreshAndPersist]
    [interval=dd:hh:mm:ss]
    [retry=[<retry interval> <# of retries>]+]
    searchbase=<base DN>
    [filter=<filter str>]
    [scope=sub|one|base]
    [attrs=<attr list>]
    [attrsonly]
    [sizelimit=<limit>]
    [timelimit=<limit>]
    [schemachecking=on|off]
    [bindmethod=simple|sasl]
    [binddn=<DN>]
    [saslmech=<mech>]
    [authcid=<identity>]
    [authzid=<identity>]
    [credentials=<passwd>]
    [realm=<realm>]
    [secprops=<properties>]
    [starttls=yes|critical]
    [tls_cert=<file>]
    [tls_key=<file>]
    [tls_cacert=<file>]
    [tls_cacertdir=<path>]
    [tls_reqcert=never|allow|try|demand]
    [tls_ciphersuite=<ciphers>]
    [tls_crlcheck=none|peer|all]
    [logbase=<base DN>]
    [logfilter=<filter str>]
    [syncdata=default|accesslog|changelog]
```
该指令通过建立当前的slapd（8）运行的syncrepl复制引擎复制消费者网站指定当前数据库作为主内容的副本。主数据库位于由提供参数指定的复制提供站点。副本数据库随时保持最新与使用LDAP内容同步协议主内容。有关该协议的更多信息，请参见RFC4533。

在摆脱参数用于复制消费者服务器内的当前的syncrepl指令，其中`<replica ID>`唯一地标识由当前的syncrepl指令所述的syncrepl规范的鉴定。 `<replica ID>`是非负的，是长度不超过三个十进制的数字。

provider参数指定包含主内容作为一个LDAP URI复制提供者站点。提供者参数指定的方案中，主机和任选其中所述提供商的slapd实例可以发现一个端口。可用于`<hostname>`是一个域名或IP地址。例如LDAP：//provider.example.com：389或LDAPS：//192.168.1.1：636。如果`<port>`没有给出，则使用标准的LDAP端口号（389或636）。注意syncrepl使用消费者发起协议，因此它的规范位于消费者的网站，而复制品规范位于提供商的网站。的syncrepl和replica指令定义了两个独立的复制机制。它们并不代表彼此的复制体。

所述的syncrepl副本的内容是使用作为其结果集的搜索规范中定义。消费者的slapd将根据搜索规范发送搜索请求给提供的slapd。搜索规范包括searchbase，范围，过滤器，ATTRS，attrsonly，的sizeLimit和时限参数，在正常的搜索规范。该searchbase参数没有默认值，必须指定。范围默认为子，过滤器默认为（objectclass=*），ATTRS默认为“*，+”复制所有的用户和业务属性，attrsonly默认情况下未设置。双方的sizeLimit和时限默认为“无限制”，只有正整数或“无限制”可能被指定。

LDAP内容同步协议有两种操作类型：refreshOnly和refreshAndPersist。操作类型由类型参数指定。在refreshOnly操作，下一个同步搜索操作周期的间隔时间改期每次同步操作结束后。间隔由间隔参数指定。它被设置为一天默认。在refreshAndPersist操作，同步搜索仍然是供应商的slapd实例持久。进一步更新主副本将产生searchResultEntry消费者的slapd以持续同步搜索的搜索响应。

如果在复制过程中发生错误时，消费者将尝试根据重试参数这是`<retry interval>`和`<# of retries>`对的列表重新连接。例如，重试=“60 10 300 3”让消费者重试每隔60秒的第10次，然后重试每300秒重试停先下一三次。`+`在`<# of retries>`中指重试，直到成功的数目不定。

模式检查可以在LDAP同步消费者现场通过接通schemachecking参数来执行。如果它被接通时，每个复制条目将被检查其模式作为条目存储到副本的内容。在副本的每个条目应该包含的架构定义所需的属性。如果它被关闭，参赛作品将被保存而不检查模式的一致性。默认是关闭的。

在指定binddn参数给出了DN绑定作为搜索的syncrepl的提供者的slapd。它应是有读权限在master数据库中复制内容的DN。

该bindmethod是简单或SASL，取决于简单的基于密码的认证或SASL认证是否是连接时的提供者的slapd实例中使用。

简单认证不应该使用，除非有充分的数据完整性和机密性的保护措施（如TLS或IPsec）。简单认证需要指定binddn和credentials参数。

一般推荐SASL认证。 SASL认证需要使用saslmech参数的机制规范。根据不同的机制，认证身份和/或可使用authcid和证书，分别被指定的凭据。所述authzid参数可以被用来指定一个授权的身份。

realm参数指定了一定的验证机制中的身份的境界。该secprops参数指定赛勒斯SASL安全属性。

启动TLS参数指定使用扩展操作认证的供应商之前建立一个TLS会话启动TLS的。如果关键参数提供，如果启动TLS请求失败的会话将被中止。否则的syncrepl会话继续不使用TLS。该TLS_REQCERT设置默认为“需求”和其他TLS设置默认为相同的主slapd的TLS设置。

而不是复制整个项目，消费者可查询的数据修改日志。这种操作模式被称为增量的syncrepl。除了上述参数，logbase和logfilter参数必须被适当地为将要使用的日志设定。所述syncdata参数必须被设置为“ACCESSLOG”如果日志符合slapo-ACCESSLOG（5）日志格式，或“更改日志”如果日志符合过时更改日志格式。如果省略或设置为“默认”的syncdata参数则日志参数将被忽略。

该的syncrepl复制机制是由BDB和HDB后端支持。

请参见本指南的LDAP同步复制一章关于如何使用此指令的更多信息。

#### 8.`updateref: <URL>`
该指令只适用于一个附属的slapd。它指定的URL返回给客户端提交其在副本更新请求。如果指定了多次，每个URL都要提供。
例如：
```
updateref:   ldap://master.example.net
```

### 4.BDB和HDB数据库指令
类别中的指令仅适用于该BDB和HDB数据库。也就是说，他们必须在“database bdb”或“database hdb”行之后，在任何后续的“backend”或“database”行之前。对于BDB/ HDB配置指令的完整说明，请参阅slapd-bdb（5）。

#### 1.`directory: <directory>`
该指令指定包含数据库的BDB文件和相关的索引所在的目录。
默认：
```
directory: /usr/local/var/openldap-data
```

## 3.配置文件示例
下面是一个示例配置文件，中间穿插了一些说明文字。它定义了两个数据库来处理X.500树的不同部分;两者都是BDB数据库实例。行号仅作参考，实际文件并不包含。首先，全局配置部分：
```
  1.    # example config file - global configuration section
  2.    include /usr/local/etc/schema/core.schema
  3.    referral ldap://root.openldap.org
  4.    access to * by * read
```
第1行是一条注释。第2行包含其他的核心模式定义的配置文件。第3行的referral指令意思为查询不是本地定义的数据库，将被称为在主机root.openldap.org的标准端口（389）上运行的LDAP服务器。

第4行是一个全局的访问控制。它适用于所有条目（任何适用的特定数据库的访问控制）。

配置文件的下一部分定义了一个在树“dc=example,dc=com”上处理事情查询的BDB后端。该数据库被复制到两个附属slapd，一个在truelies，另外一个在judgmentday。索引将会保持几个属性，userPassword属性是被保护免受未授权的访问。
```
  5.    # BDB definition for the example.com
  6.    database bdb
  7.    suffix "dc=example,dc=com"
  8.    directory /usr/local/var/openldap-data
  9.    rootdn "cn=Manager,dc=example,dc=com"
 10.    rootpw secret
 11.    # indexed attribute definitions
 12.    index uid pres,eq
 13.    index cn,sn pres,eq,approx,sub
 14.    index objectClass eq
 15.    # database access control definitions
 16.    access to attrs=userPassword
 17.        by self write
 18.        by anonymous auth
 19.        by dn.base="cn=Admin,dc=example,dc=com" write
 20.        by * none
 21.    access to *
 22.        by self write
 23.        by dn.base="cn=Admin,dc=example,dc=com" write
 24.        by * read
```
第5行是一条注释。第6行是由数据库关键字定义的数据库开始标志。第7行规定了查询该数据库的DN后缀。第8行指定该数据库文件所在的目录。

第9和10行标识数据库超级用户输入和相关联的密码。该条目不受访问控制，大小或时间限制的限制。

第12至14行表示保持对各种属性的索引。

第16至24行指定了数据库条目的访问控制。对于所有适用的条目，userPassword属性是该项本身以及由“admin”写入。它可以用于认证/授权，但在其他时候无法读取。所有其它属性都是由项和“admin”项写入，但可能会被所有用户（无论认证与否）读取。

该示例配置文件的下一部分定义了另一个BDB数据库。这一个处理涉及dc=example,dc=net子树查询，但由相同的实体作为第一数据库来管理。请注意，如果没有第39行，读访问将由于在第4行全局访问规则而被允许。
```
 33.    # BDB definition for example.net
 34.    database bdb
 35.    suffix "dc=example,dc=net"
 36.    directory /usr/local/var/openldap-data-net
 37.    rootdn "cn=Manager,dc=example,dc=com"
 38.    index objectClass eq
 39.    access to * by users read
```
# 7.运行slapd
slapd（8）被设计为一个独立运行的服务。这使得服务器能够利用缓存，管理底层数据库的并发问题，并节省系统资源。从inetd（8）运行并不是一个可选项。

# 1.命令行选项
slapd（8）在手册中详细说明了一些命令行选项。本节详细介绍了一些常用的选项。
```
-f <filename>
```
该选项指定slapd的备用配置文件。缺省通常是/usr/local/etc/openldap/slapd.conf。
```
 -F <slapd-config-directory>
```
如果同时指定-f和-F，配置文件将被读取并转换到config目录格式并写入到指定的目录。如果没有指定选项，slapd的将尝试从默认config目录读取默认的配置文件。如果一个有效的config目录存在，则默认的配置文件将被忽略。所有的slap工具适用上面的规则。

```
-h <URLs>
```
该选项指定监听配置。默认值是LDAP：///，这意味着LDAP监听所有的TCP接口的LDAP端口389。您可以指定特定的“主机/端口”对，或其他协议方案（如LDAPS：//或ldapi：//）。

|URL        |Protocol       |Transport              |
|:--        |:--            |:--                    |
|ldap:///   |LDAP           |TCP port 389           |
|ldaps:///  |LDAP over SSL  |TCP port 636           |
|ldapi:///  |LDAP           |IPC(Unix-domain socker)|

例如，-h "ldaps:// ldap://127.0.0.1:666"将创建两个监听器：一个用于（非标）ldaps:// 监听所有接口的ldaps://端口636，一个用于标准ldap://监听本地（环回）接口的666端口。Hosts指定使用主机名或IPv4或IPv6地址。端口值必须是数字。

对使用IPC的LDAP，Unix域套接字的路径名可以在URL编码。需要注意的是目录分隔符必须是URL编码，像URL的任何其他字符一样。因此，socket /usr/local/var/ldapi必须编码为
```
ldapi://%2Fusr%2Flocal%2Fvar%2Fldapi
```
ldapi：使用IPC机制的LDAP的详细使用请参阅[Chu-LDAPI]

> 注意，LDAP:///传输没有被广泛实施：非OpenLDAP的客户端可能无法使用它。

```
-n <service-name>
```
该选项指定用于记录及其他用途的服务名称。默认服务名称的slapd。

```
-l <syslog-local-user>
```
该选项指定本地用户的syslog（8）等级。值可以是LOCAL0，LOCAL1，LOCAL2，...，到LOCAL7。默认值是LOCAL4。此选项可能无法在所有系统上的支持。

```
-u user -g group
```
这些选项分别指定用户和组。用户可以是用户名或UID。组可以是组名称或GID。

```
-r directory
```
该选项指定运行时目录。slapd在开始监听之后，读取配置文件或初始化任何后端之前，将会chroot(2)该目录。

```
-d <level> | ?
```
该选项设置slapd的调试级别为<level>。当级别是“？”字符，调试级别的帮助信息将会打印，并且不管你给它的任何其他选项，slapd将会退出。当前的调试级别有

表７.1 调试级别

| 级别  | 关键字        |  描述                                |
| :---- | :----         | :----                                |
| 0     |               | 不调试                               |
| 1     | (0x1 trace)   | 跟踪函数调用                         |
| 2     | (0x2 packets) | 调试包的处理                         |
| 4     | (0x4 args)    | 重度跟踪调试                         |
| 8     | (0x8 conns)   | 连接管理                             |
| 16    | (0x10 BER)    | 打印发送和接收的包                   |
| 32    | (0x20 filter) | 搜索过滤器的处理                     |
| 64    | (0x40 config) | 配置处理                             |
| 128   | (0x80 ACL)    | 访问控制列表处理                     |
| 256   | (0x100 stats) | 连接/操作/结果的统计日志             |
| 512   | (0x200 stats2)| 发送的条目的统计日志                 |
| 1024  | (0x400 shell) | 打印和shell后端的通信                |
| 2048  | (0x800 parse) | 打印条目解析调试                     |
| 16384 | (0x4000 sync) | syncrepl消费者处理                   |
| 32768 | (0x8000 none) | 只显示那些不受日志级别设置影响的消息 |

你可以通过一次为每个需要的级别指定调试选项来启用多个级别。或者，因为调
试级别是累加的，你可以自己算一算。也就是说，如果要跟踪函数调用和查看被处理的配置文件，可以设置这两个等级的总和（在此情况下，-d65）。或者，你可以让slapd自己算，（如-d1-d64）。查看`<ldap_log.h>`了解更多详情。

> 注：slapd必须在编译时使用--enable-debug选项，在两个状态等级上的调试信息为可用的（默认值）。

# 2.运行slapd
通常slapd像下面这样运行：
```
/usr/local/libexec/slapd [<option>]*
```
/usr/local/libexec目录是由配置和上面的选项（或slapd（8））`<option>`来确定的。除非你指定一个调试级别（包括级别0），slapd将自动从其控制终端fork并在后台运行。

# 3.停止slapd
安全的关闭slapd(8)，你应该使用下面的命令：
```
kill -INT `cat /usr/local/var/slapd.pid`
```
/usr/local/var目录是由配置决定的。

通过更激烈的方法杀死的slapd可能造成信息丢失或数据库损坏。

# 8.访问控制

# 1.介绍
作为目录获取具有不同的灵敏度，控制各种授予目录访问的越来越多的数据填充变得越来越重要。例如，目录可能包含机密性质，你可能需要通过合同或法律来保护数据。或者，如果使用目录来控制访问其他服务，该目录不适当的访问可以创建导致对资产造成毁灭性的破坏攻击您的网站安全的途径。

访问到目录可以通过两种方法中，使用slapd配置文件和使用的slapd-config（5）格式（配置slapd）。

默认的访问控制策略是允许所有客户端读取。无论是什么定义访问控制策略，rootdn就总是允许充分的权利（即身份验证，搜索，比较，读，写）的一切，任何东西。

因此，它是无用的（并且在性能下降），明确列出了`<by>`子句中的rootdn。

下面的章节将介绍访问控制列表更深入和遵循一些例子和建议。完整的细节，请参见slapd.access（5）。

# 2.静态配置访问控制
以实体和属性的访问是由访问配置文件指令的控制。接入线路的一般形式为：
```
<access directive> ::= access to <what>
    [by <who> [<access>] [<control>] ]+
<what> ::= * |
    [dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
    [filter=<ldapfilter>] [attrs=<attrlist>]
<basic-style> ::= regex | exact
<scope-style> ::= base | one | subtree | children
<attrlist> ::= <attr> [val[.<basic-style>]=<regex>] | <attr> , <attrlist>
<attr> ::= <attrname> | entry | children
<who> ::= * | [anonymous | users | self
        | dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
    [dnattr=<attrname>]
    [group[/<objectclass>[/<attrname>][.<basic-style>]]=<regex>]
    [peername[.<basic-style>]=<regex>]
    [sockname[.<basic-style>]=<regex>]
    [domain[.<basic-style>]=<regex>]
    [sockurl[.<basic-style>]=<regex>]
    [set=<setspec>]
    [aci=<attrname>]
<access> ::= [self]{<level>|<priv>}
<level> ::= none | disclose | auth | compare | search | read | write | manage
<priv> ::= {=|+|-}{m|w|r|s|c|x|d|0}+
<control> ::= [stop | continue | break]
```
其中`<what>`部分选择的条目或属性来访问适用的`<who>`部分指定哪些实体授予访问权，而`<access>`部分指定授予的访问权限。多个`<who>``<access>` `<control>`三个部分的支持，这让很多实体授予同一组条目和属性的不同访问权限。不是所有的都在这里描述了这些访问控制选项;有关详细信息，请参见slapd.access（5）手册页。

### 1.用什么控制访问
访问规范的`<what>`部分确定条目和属性的访问控制适用。通过DN和过滤：报名方式有两种常见的选择。下面通过预选赛DN选择条目：
```
to *
to dn[.<basic-style>]=<regex>
to dn.<scope-style>=<DN>
```
第一种形式是用于选择所有项。第二种形式可以用于通过匹配对目标条目的DN标准化正则表达式来选择条目。（第二种形式本文档中没有进一步讨论。）第三形式用于选择哪个是DN请求的范围之内的条目。在`<DN>`是专有名称的字符串表示，如RFC4514中描述。

范围可以是base,one,subtree,children。其中，base与配置DN的条目相匹配，one匹配的父亲是配置DN的条目，subtree的匹配所有根是配置DN的子树的条目，children匹配配置DN下的所有条目（但不包括配置DN）。

例如，如果目录包含条目命名为：
```
0: o=suffix
1: cn=Manager,o=suffix
2: ou=people,o=suffix
3: uid=kdz,ou=people,o=suffix
4: cn=addresses,uid=kdz,ou=people,o=suffix
5: uid=hyc,ou=people,o=suffix
```
这样的话：
```
dn.base="ou=people,o=suffix" match 2;
dn.one="ou=people,o=suffix" match 3, and 5;
dn.subtree="ou=people,o=suffix" match 2, 3, 4, and 5; and
dn.children="ou=people,o=suffix" match 3, 4, and 5.
```
条目还可以使用过滤器选择：
```
to filter=<ldap filter>
```
其中，`<ldap filter>`是LDAP搜索筛选器的字符串，如RFC4515。例如：
```
to filter=(objectClass=person)
```
注意，条目可以由DN选择和过滤器通过`<what>`子句进行选择。
```
to dn.one="ou=people,o=suffix" filter=(objectClass=person)
```
条目的属性是从一个包括属性名称的由逗号分隔的`<what>`选择器列表中选择的：
```
attrs=<attribute list>
```
属性的具体值是通过使用单一的属性名和值的选择器选择的：
```
attrs=<attribute> val[.<style>]=<regex>
```
有两个特殊的伪属性条目和children。要阅读（因而返回）目标条目中，主体必须具有读取目标条目属性的权限。为了执行搜索，主体必须具有对搜索库的入口属性搜索的权限。要添加或删除条目，主体必须具有对条目的属性的写权限，并且必须有写权限的条目的父母的儿子属性。要重命名条目，主体必须具有条目的条目属性的写权限并拥有原来的父母和新父母的儿子属性的写权限。在本节末尾的完整例子应能帮助你清楚的了解所有事情。

最后，还有一个特殊的条目选择器`"*"`用来选择任何条目。当没有提供其他的`<what>`选择器时使用。它相当于`"dn=.*"`。

### 2.谁授予访问权限
在`<who>`部分标识的实体被授予访问权限。注意，访问权限授予的"entities"而不是"entries"。下表总结了实体标示符：

表6.3 访问实体标示符

| 说明                         | 实体                       |
| :-                           | :--                        |
| *                            | 所有的，包括匿名和认证用户 |
| 匿名                         | 匿名（未经身份验证）用户   |
| 用户                         | 认证用户                   |
| 自己                    　   | 用户与目标条目关联　　     |
| `dn[.<basic-style>]=<regex>` | 用户匹配一个正则表达式     |
| `dn.<scope-style>=<DN>`      | 一个DN范围内的用户         |

该DN标示符的行为与`<what>`子句的DN标示符很像。

其他的控制因素也支持。例如，`<who>`可以被通过在向其中接入应用项在一个DN值属性中列出的条目的限制：
```
dnattr=<dn-valued attribute name>
```
该dnattr规范是用来访问DN的条目属性被列出的条目（例如，访问一组被列为组条目的所有者的条目）。

有些因素可能并不适用于所有的（或任何）环境。例如，域名因素依赖于IP，以域名查找。因为这些可以容易地伪造，域因子应当避免。

### 3.访问授予
`<access>`授权的种类可以是下列之一：

表6.4 访问等级

| 等级        | 授权    | 描述              |
| :----       |-----:   | :----             |
| none =      | 0       | 无权访问          |
| disclose =  | d       | 需要错误信息披露  |
| auth =      | dx      | 需要验证（绑定）  |
| compare =   | cdx     | 需要比较          |
| search =    | scdx    | 需要接受搜索条件  |
| read =      | rscdx   | 需要读取搜索结果  |
| write =     | wrscdx  | 需要修改或重命名  |
| manage =    | mwrscdx | 需要管理          |

每个级别都可以访问的比它低的级别。因此，例如，授予某人写访问的条目还将授予他读，搜索，比较，认证和披露的访问权限。然而，人们可以使用权限说明符授予特定的权限。

### 4.访问控制评估
评估某个请求者是否应该被给予访问一个条目或属性的权限，slapd会比较条目或属性与配置文件中给出的`<what>`选择器。对于每个条目，首先保持该条目（或者如果在任何数据库中未保持的全局访问指令）数据库提供的访问控制，随后保持全局访问指令。然而，随着访问列表的分配，因为全局访问列表有效地追加到每个数据库列表中，如果结果列表非空，列表将由于`access to * by * none`指令结束。如果没有访问指令适用与后端，则使用默认的读取权限。

在这个优先级，访问指令，在它们出现在配置文件中的顺序检查。 当匹配到条目或属性，SLAPD将会在第一个`<what>`选择器停止。相应的访问控制指令就是slapd将要评估访问的指令。

接下来，slapd比较实体请求与上述顺序出现的访问指令的`<who>`选择器。当匹配到请求者，它将会在第一个`<who>`选择器停止。这决定了访问的实体具有对条目或属性的访问权限。

最后，slapd的比较访问权限与由客户端访问请求的`<access>`选择器。如果它允许大于或等于访问，则允许访问。否则，访问被拒绝。

存取控制指令的计算顺序使得它们在配置文件中的位置非常重要。如果一个访问指令在它选择的条目中比另一条指令更具体，它应该首先出现在配置文件中。同样，如果一个`<who>`选择器比另一个更具体，它应该首先存取控制指令。下面给出的访问控制例子应能帮助说明这一点。

### 5.访问控制示例
上面描述的访问控制功能是相当强大的。该部分显示具体使用的一些例子。

一个简单的例子：
```
access to * by * read
```
这条指令将读取权限授权给所有人。
```
access to *
    by self write
    by anonymous auth
    by * read
```
该指令允许用户修改自己的条目，允许匿名身份验证这些条目，并允许所有其他人阅读这些条目。请注意，只有先通过它匹配适用`<who>`子句。因此，匿名用户被授予身份验证，但没有读取权限。最后一节也可以写为"by users read"。

它通常希望限制保护到位级操作。以下说明如何使用安全强度因子（SSF）。
```
access to *
    by ssf=128 self write
    by ssf=64 anonymous auth
    by ssf=64 users read
```
该指令允许用户修改自己的条目，当安全级别在128或更高时；允许匿名用户访问身份验证，允许所有用户获得读权限，当安全级别在64或更高时。如果客户没有充分建立安全保护，默认的`by * none`将被使用。

下面的例子演示了使用样式符的两个访问指令，其中排序是通过DN条目的显著选择。
```
access to dn.children="dc=example,dc=com"
    by * search
access to dn.children="dc=com"
    by * read
```
读访问权限将被授权给所有`dc=com`的子树，搜索权限将被授权给所有`dc=example,dc=com`的子树。如果没有访问指令匹配DN`dc=com`，则将不会被授权。如果这些访问指令的顺序相反，尾部指令将永远不会到达，因为`dc=example,dc=com`下的所有条目也都在`dc=com`之下。

还要注意的是，如果没有由指令匹配的访问或者没有`<who>`子句，**访问将被拒绝**。也就是说，指令每次访问都将由一个隐式的`by * none`结束。当使用访问列表处理，全局访问列表将有效地追加到每个数据库列表中，如果结果列表非空，列表将由`access to * by * none`指令结束。如果没有访问指令适用与后端，则使用默认的读取权限。

在下一个例子中再次展示了访问指令和`<who>`子句排序的重要性。它也显示了使用属性选择的访问权限授予特定属性和各种`<who>`选择器。
```
access to dn.subtree="dc=example,dc=com" attrs=homePhone
    by self write
    by dn.children="dc=example,dc=com" search
    by peername.regex=IP:10\..+ read
access to dn.subtree="dc=example,dc=com"
    by self write
    by dn.children="dc=example,dc=com" search
    by anonymous auth
```
此示例适用于条目中的"dc=example,dc=com"子树。对于除了homePhone外的所有的属性，一个条目可以写自己，在example.com下的项都可以被搜索，其他任何人都访问（隐式`by * none`）而无需认证/授权（总是匿名的）。homePhone属性可以被任何条目写，被example.com下的条目搜索，被通过网络段"10"开头的连接的客户读，其他的都不可读（隐式`by * none`）。所有其他访问是被拒绝，隐式`access to * by * none`。

有时允许特定的DN来添加或从属性中删除自身是有用。例如，如果你想创建一个组，让人们从成员属性中添加和删除自己的DN，可以用这样的访问指令完成它：
```
access to attrs=member,entry
    by dnattr=member selfwrite
```
这个dnattr`<who>`选择器的意思是，访问适用于成员属性中的条目列表。selfwrite的意思是，这些成员只能从属性中添加或删除自己的DN，而不是其他的值。条目属性的添加，因为需要访问条目来访问任何一个条目的属性是必需的。

## 3.动态配置访问控制
olcAccess属性控制访问slapd的条目和属性，其值是顺序的访问指令。在olcAccess配置的一般形式是：
```
olcAccess: <access directive>
<access directive> ::= to <what>
    [by <who> [<access>] [<control>] ]+
<what> ::= * |
    [dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
    [filter=<ldapfilter>] [attrs=<attrlist>]
<basic-style> ::= regex | exact
<scope-style> ::= base | one | subtree | children
<attrlist> ::= <attr> [val[.<basic-style>]=<regex>] | <attr> , <attrlist>
<attr> ::= <attrname> | entry | children
<who> ::= * | [anonymous | users | self
        | dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
    [dnattr=<attrname>]
    [group[/<objectclass>[/<attrname>][.<basic-style>]]=<regex>]
    [peername[.<basic-style>]=<regex>]
    [sockname[.<basic-style>]=<regex>]
    [domain[.<basic-style>]=<regex>]
    [sockurl[.<basic-style>]=<regex>]
    [set=<setspec>]
    [aci=<attrname>]
<access> ::= [self]{<level>|<priv>}
<level> ::= none | disclose | auth | compare | search | read | write | manage
<priv> ::= {=|+|-}{m|w|r|s|c|x|d|0}+
<control> ::= [stop | continue | break]
```
其中`<what>`部分选择的条目或属性来访问适用的`<who>`部分指定哪些实体授予访问权，而`<access>`部分指定授予的访问权限。多个`<who>``<access>` `<control>`三个部分的支持，这让很多实体授予同一组条目和属性的不同访问权限。不是所有的都在这里描述了这些访问控制选项;有关详细信息，请参见slapd.access（5）手册页。

### 1.用什么控制访问
访问规范的`<what>`部分确定条目和属性的访问控制适用。通过DN和过滤：报名方式有两种常见的选择。下面通过预选赛DN选择条目：
```
to *
to dn[.<basic-style>]=<regex>
to dn.<scope-style>=<DN>
```
第一种形式是用于选择所有项。第二种形式可以用于通过匹配对目标条目的DN标准化正则表达式来选择条目。（第二种形式本文档中没有进一步讨论。）第三形式用于选择哪个是DN请求的范围之内的条目。在`<DN>`是专有名称的字符串表示，如RFC4514中描述。

范围可以是base,one,subtree,children。其中，base与配置DN的条目相匹配，one匹配的父亲是配置DN的条目，subtree的匹配所有根是配置DN的子树的条目，children匹配配置DN下的所有条目（但不包括配置DN）。

例如，如果目录包含条目命名为：
```
0: o=suffix
1: cn=Manager,o=suffix
2: ou=people,o=suffix
3: uid=kdz,ou=people,o=suffix
4: cn=addresses,uid=kdz,ou=people,o=suffix
5: uid=hyc,ou=people,o=suffix
```
这样的话：
```
dn.base="ou=people,o=suffix" match 2;
dn.one="ou=people,o=suffix" match 3, and 5;
dn.subtree="ou=people,o=suffix" match 2, 3, 4, and 5; and
dn.children="ou=people,o=suffix" match 3, 4, and 5.
```
条目还可以使用过滤器选择：
```
to filter=<ldap filter>
```
其中，`<ldap filter>`是LDAP搜索筛选器的字符串，如RFC4515。例如：
```
to filter=(objectClass=person)
```
注意，条目可以由DN选择和过滤器通过`<what>`子句进行选择。
```
to dn.one="ou=people,o=suffix" filter=(objectClass=person)
```
条目的属性是从一个包括属性名称的由逗号分隔的`<what>`选择器列表中选择的：
```
attrs=<attribute list>
```
属性的具体值是通过使用单一的属性名和值的选择器选择的：
```
attrs=<attribute> val[.<style>]=<regex>
```
有两个特殊的伪属性条目和children。要阅读（因而返回）目标条目中，主体必须具有读取目标条目属性的权限。为了执行搜索，主体必须具有对搜索库的入口属性搜索的权限。要添加或删除条目，主体必须具有对条目的属性的写权限，并且必须有写权限的条目的父母的儿子属性。要重命名条目，主体必须具有条目的条目属性的写权限并拥有原来的父母和新父母的儿子属性的写权限。在本节末尾的完整例子应能帮助你清楚的了解所有事情。

最后，还有一个特殊的条目选择器`"*"`用来选择任何条目。当没有提供其他的`<what>`选择器时使用。它相当于`"dn=.*"`。

### 2.谁授予访问权限
在`<who>`部分标识的实体被授予访问权限。注意，访问权限授予的"entities"而不是"entries"。下表总结了实体标示符：

表6.3 访问实体标示符

| 说明                         | 实体                       |
| :-                           | :--                        |
| *                            | 所有的，包括匿名和认证用户 |
| 匿名                         | 匿名（未经身份验证）用户   |
| 用户                         | 认证用户                   |
| 自己                    　   | 用户与目标条目关联　　     |
| `dn[.<basic-style>]=<regex>` | 用户匹配一个正则表达式     |
| `dn.<scope-style>=<DN>`      | 一个DN范围内的用户         |

该DN标示符的行为与`<what>`子句的DN标示符很像。

其他的控制因素也支持。例如，`<who>`可以被通过在向其中接入应用项在一个DN值属性中列出的条目的限制：
```
dnattr=<dn-valued attribute name>
```
该dnattr规范是用来访问DN的条目属性被列出的条目（例如，访问一组被列为组条目的所有者的条目）。

有些因素可能并不适用于所有的（或任何）环境。例如，域名因素依赖于IP，以域名查找。因为这些可以容易地伪造，域因子应当避免。

### 3.访问授予
`<access>`授权的种类可以是下列之一：

表6.4 访问等级

| 等级        | 授权    | 描述              |
| :----       | ----:   | :----             |
| none =      | 0       | 无权访问          |
| disclose =  | d       | 需要错误信息披露  |
| auth =      | dx      | 需要验证（绑定）  |
| compare =   | cdx     | 需要比较          |
| search =    | scdx    | 需要接受搜索条件  |
| read =      | rscdx   | 需要读取搜索结果  |
| write =     | wrscdx  | 需要修改或重命名  |
| manage =    | mwrscdx | 需要管理          |

每个级别都可以访问的比它低的级别。因此，例如，授予某人写访问的条目还将授予他读，搜索，比较，认证和披露的访问权限。然而，人们可以使用权限说明符授予特定的权限。

### 4.访问控制评估
评估某个请求者是否应该被给予访问一个条目或属性的权限，slapd会比较条目或属性与配置文件中给出的`<what>`选择器。对于每个条目，首先保持该条目（或者如果在任何数据库中未保持的全局访问指令）数据库提供的访问控制，随后保持全局访问指令。然而，随着访问列表的分配，因为全局访问列表有效地追加到每个数据库列表中，如果结果列表非空，列表将由于`access to * by * none`指令结束。如果没有访问指令适用与后端，则使用默认的读取权限。

在这个优先级，访问指令，在它们出现在配置文件中的顺序检查。 当匹配到条目或属性，SLAPD将会在第一个`<what>`选择器停止。相应的访问控制指令就是slapd将要评估访问的指令。

接下来，slapd比较实体请求与上述顺序出现的访问指令的`<who>`选择器。当匹配到请求者，它将会在第一个`<who>`选择器停止。这决定了访问的实体具有对条目或属性的访问权限。

最后，slapd的比较访问权限与由客户端访问请求的`<access>`选择器。如果它允许大于或等于访问，则允许访问。否则，访问被拒绝。

存取控制指令的计算顺序使得它们在配置文件中的位置非常重要。如果一个访问指令在它选择的条目中比另一条指令更具体，它应该首先出现在配置文件中。同样，如果一个`<who>`选择器比另一个更具体，它应该首先存取控制指令。下面给出的访问控制例子应能帮助说明这一点。

### 5.访问控制示例
上面描述的访问控制功能是相当强大的。该部分显示具体使用的一些例子。

一个简单的例子：
```
olcAccess to * by * read
```
这条指令将读取权限授权给所有人。
```
olcAccess to *
    by self write
    by anonymous auth
    by * read
```
该指令允许用户修改自己的条目，允许匿名身份验证这些条目，并允许所有其他人阅读这些条目。请注意，只有先通过它匹配适用`<who>`子句。因此，匿名用户被授予身份验证，但没有读取权限。最后一节也可以写为"by users read"。

它通常希望限制保护到位级操作。以下说明如何使用安全强度因子（SSF）。
```
olcAccess to *
    by ssf=128 self write
    by ssf=64 anonymous auth
    by ssf=64 users read
```
该指令允许用户修改自己的条目，当安全级别在128或更高时；允许匿名用户访问身份验证，允许所有用户获得读权限，当安全级别在64或更高时。如果客户没有充分建立安全保护，默认的`by * none`将被使用。

下面的例子演示了使用样式符的两个访问指令，其中排序是通过DN条目的显著选择。
```
olcAccess to dn.children="dc=example,dc=com"
    by * search
olcAccess to dn.children="dc=com"
    by * read
```
读访问权限将被授权给所有`dc=com`的子树，搜索权限将被授权给所有`dc=example,dc=com`的子树。如果没有访问指令匹配DN`dc=com`，则将不会被授权。如果这些访问指令的顺序相反，尾部指令将永远不会到达，因为`dc=example,dc=com`下的所有条目也都在`dc=com`之下。

还要注意的是，如果没有由指令匹配的访问或者没有`<who>`子句，**访问将被拒绝**。也就是说，指令每次访问都将由一个隐式的`by * none`结束。当使用访问列表处理，全局访问列表将有效地追加到每个数据库列表中，如果结果列表非空，列表将由`access to * by * none`指令结束。如果没有访问指令适用与后端，则使用默认的读取权限。

在下一个例子中再次展示了访问指令和`<who>`子句排序的重要性。它也显示了使用属性选择的访问权限授予特定属性和各种`<who>`选择器。
```
olcAccess to dn.subtree="dc=example,dc=com" attrs=homePhone
    by self write
    by dn.children="dc=example,dc=com" search
    by peername.regex=IP:10\..+ read
olcAccess to dn.subtree="dc=example,dc=com"
    by self write
    by dn.children="dc=example,dc=com" search
    by anonymous auth
```
此示例适用于条目中的"dc=example,dc=com"子树。对于除了homePhone外的所有的属性，一个条目可以写自己，在example.com下的项都可以被搜索，其他任何人都访问（隐式`by * none`）而无需认证/授权（总是匿名的）。homePhone属性可以被任何条目写，被example.com下的条目搜索，被通过网络段"10"开头的连接的客户读，其他的都不可读（隐式`by * none`）。所有其他访问是被拒绝，隐式`access to * by * none`。

有时允许特定的DN来添加或从属性中删除自身是有用。例如，如果你想创建一个组，让人们从成员属性中添加和删除自己的DN，可以用这样的访问指令完成它：
```
olcAccess to attrs=member,entry
    by dnattr=member selfwrite
```
这个dnattr`<who>`选择器的意思是，访问适用于成员属性中的条目列表。selfwrite的意思是，这些成员只能从属性中添加或删除自己的DN，而不是其他的值。条目属性的添加，因为需要访问条目来访问任何一个条目的属性是必需的。

### 6.访问控制排序
由于olcAccess指令的顺序对他们正确的评估至关重要，但通常LDAP属性不保留它们的值的顺序，OpenLDAP的使用自定义schema扩展来维护这些固定的值排序。这个排序在前面加上“{X}”数字索引到的每个值，用于订购的配置项的方法类似于维护。这些索引标签被slapd自动维护，不需要指定原本定义的值。例如，当您创建的设置如下：
```
olcAccess: to attrs=member,entry
    by dnattr=member selfwrite
olcAccess: to dn.children="dc=example,dc=com"
    by * search
olcAccess: to dn.children="dc=com"
    by * read
```
当你使用slapcat或ldapsearch查看他们时，它们将包含：
```
olcAccess: {0}to attrs=member,entry
    by dnattr=member selfwrite
olcAccess: {1}to dn.children="dc=example,dc=com"
    by * search
olcAccess: {2}to dn.children="dc=com"
    by * read
```
数字索引可被用于指定一个特定的值，当使用ldapmodify修改访问规则时改变。这个索引可以用来代替（或附加）实际访问的值。管理多个访问规则时，使用这个数字索引是非常有用的。

例如，如果我们需要改变上述第二条规则，以允许写入访问，而不是搜索，我们可以试试这个LDIF：
```
changetype: modify
delete: olcAccess
olcAccess: to dn.children="dc=example,dc=com" by * search
-
add: olcAccess
olcAccess: to dn.children="dc=example,dc=com" by * write
-
```
但这个例子不能保证现有的值仍留在原来的顺序，因此，将最有可能对安全配置产生破坏。取而代之的是，应该使用数字索引：
```
changetype: modify
delete: olcAccess
olcAccess: {1}
-
add: olcAccess
olcAccess: {1}to dn.children="dc=example,dc=com" by * write
-
```
本示例删除任何olcAccess属性值＃1（无论价值多少）的规则，并补充说，被明确作为插入值＃1的新值。其结果将是
```
olcAccess: {0}to attrs=member,entry
    by dnattr=member selfwrite
olcAccess: {1}to dn.children="dc=example,dc=com"
    by * write
olcAccess: {2}to dn.children="dc=com"
    by * read
```
就是这样。

## 4.访问控制常见例子
### 1.基本的ACLs
一般来说,一些基本的访问控制列表的开头，如下：
```
access to attr=userPassword
    by self =xw
    by anonymous auth
    by * none

access to *
    by self write
    by users read
    by * none
```
第一个ACL允许用户更新（但不读）自己的密码，匿名用户对这个属性进行身份验证，以及（隐含的）拒绝所有其他人的访问。

第二个ACL允许他们获得用户完全访问权限，身份验证的用户读取访问任何东西，和（隐含的）拒绝所有其他人的访问（通常是匿名用户）。

### 2.匹配匿名和认证用户
匿名用户有一个空的DN。当`dn.exact=""`或`dn.regex="^$"`使用时，slapd的（8）提供了一个匿名缩写应被替用。
```
access to *
    by anonymous none
    by * read
```
拒绝所有匿名用户访问，而其他人授予读取权限。

身份验证的用户有一个对象DN。虽然`dn.regex=".+"`将匹配任何身份验证的用户，OpenLDAP的为用户提供了short hand应该被替代。
```
access to *
    by users read
    by * none
```
此ACL授予读取权限身份验证的用户，同时拒绝其他人（即：匿名用户）。

### 3.控制rootDN的访问
你可以在slapd.conf(5)中指定rootdn或者在slapd.d中不指定rootpw。然后，你必须添加具有相同DN的实际的目录项，例如：
```
dn: cn=Manager,o=MyOrganization
cn: Manager
sn: Manager
objectClass: person
objectClass: top
userPassword: {SSHA}someSSHAdata
```
然后rootdn将需要定期绑定到该DN，这又要求该条目的DN和userPassword的AUTH访问，并且这可以通过访问控制列表进行限制。例如：
```
access to dn.base="cn=Manager,o=MyOrganization"
    by peername.regex=127\.0\.0\.1 auth
    by peername.regex=192\.168\.0\..* auth
    by users none
      by * none
```
上面的ACLs只允许将rootdn从本地主机和192.168.0.0/24进行绑定。

### 4.管理组的访问
有几个方法可以做到这一点。下面说说其中一种方法。请看下面的DIT布局：
```
+-dc=example,dc=com
+---cn=administrators,dc=example,dc=com
+---cn=fred blogs,dc=example,dc=com
```
及以下组对象（LDIF格式）：
```
dn: cn=administrators,dc=example,dc=com
cn: administrators of this region
objectclass: groupOfNames  (important for the group acl feature)
member: cn=fred blogs,dc=example,dc=com
member: cn=somebody else,dc=example,dc=com
```
那么可以通过slapd.conf（5）的`by group`组访问指令授予该组的成员。例如：
```
access to dn.children="dc=example,dc=com"
    by self write
    by group.exact="cn=Administrators,dc=example,dc=com" write
    by * auth
```
dn子句中，可以使用`expand`来扩展基于目标的正则表达式匹配的组名称，也就是说，像`dn.regex`。 例如，
```
access to dn.regex="(.+,)?ou=People,(dc=[^,]+,dc=[^,]+)$"
        attrs=children,entry,uid
    by group.expand="cn=Managers,$2" write
    by users read
    by * auth
```
上面假设组成员都在成员属性groupofNames对象类中被发现。如果你需要使用不同的组对象或不同的属性类型，使用下面的slapd.conf（5）（略）语法：
```
access to <what>
    by group/<objectclass>/<attributename>=<DN> <access>
```
例如：
```
access to *
    by group/organizationalRole/roleOccupant="cn=Administrator,dc=example,dc=com" write
```
在这种情况下，我们有一个对象类organizationalRole包含在管理员DN的roleOccupant属性中。 例如：
```
dn: cn=Administrator,dc=example,dc=com
cn: Administrator
objectclass: organizationalRole
roleOccupant: cn=Jane Doe,dc=example,dc=com
```

> 注：指定的成员属性的类型必须是DN或NameAndOptionalUID语法，并指定对象类应该允许的属性类型。

动态组还支持访问控制。请参阅slapo-dynlist（5）和动态列表重叠部分。

### 5.子集属性的访问授权
你可以通过指定ACL子句中属性名称的列表，对一组属性授予访问权限。通常你还需要授予访问条目本身。还要注意如何控制children添加，删除和重命名条目的能力。
```
# mail: self may write, authenticated users may read
access to attrs=mail
  by self write
  by users read
  by * none

# cn, sn: self my write, all may read
access to attrs=cn,sn
  by self write
  by * read

# immediate children: only self can add/delete entries under this entry
access to attrs=children
  by self write

# entry itself: self may write, all may read
access to attrs=entry
  by self write
  by * read

# other attributes: self may write, others have no access
access to *
  by self write
  by * none
```
对象类名称也可以在该列表中，这会影响所有需要或由该对象类所允许的属性指定。其实，在attrlist名字由@前缀被直接当作对象类的名称。一个名称前缀为！也被视为一个对象类，但在这种情况下，访问规则将影响不需要，也不由objectClass所允许的属性。

### 6. 允许用户修改他们自己的所有条目
对于设置，用户可以修改所有自己的纪录和所有自己的孩子：
```
access to dn.regex="(.+,)?(uid=[^,]+,o=Company)$"
   by dn.exact,expand="$2" write
   by anonymous auth
```
（添加更多的例子）

### 7. 允许创建条目
比方说，你有这样的：
```
o=<basedn>
    ou=domains
        associatedDomain=<somedomain>
            ou=users
                uid=<someuserid>
                uid=<someotheruserid>
            ou=addressbooks
                uid=<someuserid>
                    cn=<someone>
                    cn=<someoneelse>
```
并且，对于另一个域`<someotherdomain>`：
```
o=<basedn>
    ou=domains
        associatedDomain=<someotherdomain>
            ou=users
                uid=<someuserid>
                uid=<someotheruserid>
            ou=addressbooks
                uid=<someotheruserid>
                    cn=<someone>
                    cn=<someoneelse>
```
那么，如果你想用户`uid=<someuserid>`仅创建自己的事项，你可以写一个这样的ACL：
```
# this rule lets users of "associatedDomain=<matcheddomain>"
# write under "ou=addressbook,associatedDomain=<matcheddomain>,ou=domains,o=<basedn>",
# i.e. a user can write ANY entry below its domain's address book;
# this permission is necessary, but not sufficient, the next
# will restrict this permission further

access to dn.regex="^ou=addressbook,associatedDomain=([^,]+),ou=domains,o=<basedn>$" attrs=children
        by dn.regex="^uid=([^,]+),ou=users,associatedDomain=$1,ou=domains,o=<basedn>$$" write
        by * none

# Note that above the "by" clause needs a "regex" style to make sure
# it expands to a DN that starts with a "uid=<someuserid>" pattern
# while substituting the associatedDomain submatch from the "what" clause.

# This rule lets a user with "uid=<matcheduid>" of "<associatedDomain=matcheddomain>"
# write (i.e. add, modify, delete) the entry whose DN is exactly
# "uid=<matcheduid>,ou=addressbook,associatedDomain=<matcheddomain>,ou=domains,o=<basedn>"
# and ANY entry as subtree of it

access to dn.regex="^(.+,)?uid=([^,]+),ou=addressbook,associatedDomain=([^,]+),ou=domains,o=<basedn>$"
        by dn.exact,expand="uid=$2,ou=users,associatedDomain=$3,ou=domains,o=<basedn>" write
        by * none

# Note that above the "by" clause uses the "exact" style with the "expand"
# modifier because now the whole pattern can be rebuilt by means of the
# submatches from the "what" clause, so a "regex" compilation and evaluation
# is no longer required.
```

### 8. 访问控制中使用正则表达式的tips
始终使用`dn.regex=<pattern>`当你打算使用正则表达式匹配。单独的`DN=<pattern>`被默认为`dn.exact<pattern>`。

当你想匹配至少一个字符用`(.+)`，而不是`(.*)`。`(.*)`匹配空字符串为好。

如果可以使用其他更安全，更便宜的方式，不要使用正则表达式。 例子：
```
dn.regex=".*dc=example,dc=com"
```
是不安全的，昂贵的：
> * 不安全的是因为含有`dc=example,dc=com`将匹配任何串，不仅是那些以你想要的模式结束的;可以使用`.*dc=example,dc=com$`来代替。
> * 不安全还因为它将使作为字符串中命名的第一个RDN属性，例如任何属性类型直流结束一个自定义的属性类型是myDC将匹配也是如此。如果你真的需要一个正则表达式，可以让刚刚`dc=example,dc=com`或任何其子树，用`^(.+,)?dc=example,dc=com$`，这意味着：任何`dc=...`，如果有的话（括号内的模式之后的问号），必须以逗号结束;
> * 昂贵的是因为如果你不需要子匹配，您可以使用范围界定的风格，例如

```
dn.subtree="dc=example,dc=com"
```
包括`dc=example,dc=com`在匹配模式中，
```
dn.children="dc=example,dc=com"
```
从匹配模式排`dc=example,dc=com`，或
```
dn.onelevel="dc=example,dc=com"
```
允许只有一个子级的匹配。

通常使用^和$的正则表达式，在适当的时候，因为`ou=(.+),ou=(.+),ou=addressbooks,o=basedn`将匹配`something=bla,ou=xxx,ou=yyy,ou=addressbooks,o=basedn,ou=addressbooks,o=basedn,dc=some,dc=org`

通常使用`([^,]+)`来表示只有一个RDN，因为`(.+)`可以包括任何数量的RDN; 例如`ou=(.+),dc=example,dc=com`将匹配`ou=My,o=Org,dc=example,dc=com`，这可能不是你想要的。

决不要通过by子句添加rootdn。ACLs不会对rootdn身份执行操作处理（否则就没有任何理由来定义rootdn）。

使用简写。用户指令与认证用户相匹配，并且匿名指令与匿名用户相匹配。

如果你需要的是确定范围或更换子串不要将`<by>`子句用于`dn.regex`格式;使用范围界定样式（例如精确，ONELEVEL，子女或子树）和修改风格扩张所导致的子串扩张。

例如，
```
access to dn.regex=".+,dc=([^,]+),dc=([^,]+)$"
  by dn.regex="^[^,],ou=Admin,dc=$1,dc=$2$$" write
```
虽然正确，可通过安全有效地代替
```
access to dn.regex=".+,(dc=[^,]+,dc=[^,]+)$"
  by dn.onelevel,expand="ou=Admin,$1" write
```
其中`<what>`子句中的正则表达式是更紧凑，一个<by>子句中通过扩展子一级的更有效的作用域的风格所取代。

### 9. 基于安全强度系数（SSF）的授权和拒绝访问
你可以根据安全强度系数（SSF）限制访问
```
access to dn="cn=example,cn=edu"
  by * ssf=256 read
```
0（零）意味着没有保障，1只意味着完整性保护，56位DES或其他弱密码，112位三重DES和其他强密码，128位 RC4，Blowfish和其他现代化的强密码。

其他可能性：
```
transport_ssf=<n>
tls_ssf=<n>
sasl_ssf=<n>
```
建议使用256位密码。
请参阅有关SSF信息的slapd.conf（5）。

### 10.当预期的东西不工作
考虑下面这个例子：
```
access to *
  by anonymous auth

access to *
  by self write

access to *
  by users read
```
你可能会认为这将允许任何用户登录，如果他登录阅读一切，改变他自己的数据。但是，在这个例子中只有登录作品和到ldapsearch返回任何数据。问题是，通过SLAPD线经过其访问的配置线，一旦发现在访问规则的一部分匹配停止。（此处为*）

为了得到我们想要的文件中有如下：
```
access to *
  by anonymous auth
  by self write
  by users read
```
一般的规律是：“特殊访问规则第一，一般的访问规则最后的”
参见slapd.access（5），用于调试信息（8）日志128级和slapacl命令。

## 5.集 - 基于关系授予权限
集能通过实例最好的展示。下面的章节将为了方便自己的理解给出了几个组ACL例子。

（参见集在访问控制中的[FAQ](http://www.openldap.org/faq/data/cache/1133.html)）

> 注意：集被认为是实验性的。

### 1.Groups of Groups
OpenLDAP的ACL组没有展开组内的组，而是作为另一组的成员组。 例如：
```
 dn: cn=sudoadm,ou=group,dc=example,dc=com
 cn: sudoadm
 objectClass: groupOfNames
 member: uid=john,ou=people,dc=example,dc=com
 member: cn=accountadm,ou=group,dc=example,dc=com

 dn: cn=accountadm,ou=group,dc=example,dc=com
 cn: accountadm
 objectClass: groupOfNames
 member: uid=mary,ou=people,dc=example,dc=com
```
如果我们使用标准组的ACL与上面的条目，并允许管理员组的成员有写权限，mary将不包括在内：
```
access to dn.subtree="ou=sudoers,dc=example,dc=com"
    by group.exact="cn=sudoadm,ou=group,dc=example,dc=com" write
    by * read
```
有了集，我们可以让ACL在组内是递归的和群体的。所以组中的每个成员，它进一步扩大：
```
access to dn.subtree="ou=sudoers,dc=example,dc=com"
    by set="[cn=sudoadm,ou=group,dc=example,dc=com]/member* & user" write
    by * read
```
这组ACL就是说，将`cn=sudoadm`的DN，检查它的成员属性（们）（其中“*”表示递归），并与交叉已验证用户的DN的结果。如果结果不为空，则ACL被认为是匹配的，写访问是理所当然的。

下图说明了组是如何构建的：

![图8.1：填充递归组集合](http://www.openldap.org/doc/admin24/set-recursivegroup.png)

图8.1：填充递归组集合

首先，我们得到的`uid=john`的DN。此项目没有一个成员属性，所以扩张到此为止。现在，我们得到`cn=accountadm`。这其中确实有一个成员属性，`uid=mary`。该`uid=mary`项，之后没有成员，所以我们在这里再次停止。最终的比较：
```
{"uid=john,ou=people,dc=example,dc=com","uid=mary,ou=people,dc=example,dc=com"} & user
```
如果验证用户的DN是这两个中的任何一个，写访问是理所当然的。所以这集将包括须sudoadm组玛丽，她将被允许写访问。

### 2.没有DN语法的组ACLS
传统的组ACL，前面的例子中甚至有递归组，要求成员指定如DNs等，而不仅仅是用户名。

有了集，它也可以使用组ACLs中的简单名称，本例将显示。

比方说，我们希望允许sudoadm组的成员写信给我们的树的`ou=suders`分支。但是，我们组定义现在使用memberUid为组成员：
```
 dn: cn=sudoadm,ou=group,dc=example,dc=com
 cn: sudoadm
 objectClass: posixGroup
 gidNumber: 1000
 memberUid: john
```
对于这种类型的组中，我们不能使用组ACLs。但是，有一组访问控制列表，我们可以授予所需的访问：
```
access to dn.subtree="ou=sudoers,dc=example,dc=com"
    by set="[cn=sudoadm,ou=group,dc=example,dc=com]/memberUid & user/uid" write
    by * read
```
我们用一个简单的方法，我们对连接（和认证）用户的uid属性和同组的memberUid属性进行比较。如果它们匹配，交集非空和ACL将允许写入访问。

下图说明了这一套，当连接的认证用户是`uid=john,ou=people,dc=example,dc=com`

![图8.2：有memberUid的Sets](http://www.openldap.org/doc/admin24/set-memberUid.png)

图8.2：有memberUid的Sets

在这种情况下，这是一个匹配。如果是mary是认证用户，但是她将被拒绝对`ou=sudoers`的写访问，因为她的uid属性没有在组memberUid中。

### 3.Following references
现在，我们将呈现使用Sets来完成一个相当强大的例子。这个例子往往使OpenLDAP的管理员微笑，在他们理解它并知道它的含义。

让我们先从一个用户条目开始：
```
 dn: uid=john,ou=people,dc=example,dc=com
 uid: john
 objectClass: inetOrgPerson
 givenName: John
 sn: Smith
 cn: john
 manager: uid=mary,ou=people,dc=example,dc=com
```
编写一个ACL，使用Sets使管理更新一些属性非常简单：
```
 access to dn.exact="uid=john,ou=people,dc=example,dc=com"
    attrs=carLicense,homePhone,mobile,pager,telephoneNumber
    by self write
    by set="this/manager & user" write
    by * read
```
在这组，这扩展到正在访问的条目，使这一/拓展经理john的入口被访问到`uid=mary,ou=people,dc=example,dc=com`。如果管理者自己正在访问john的进入，ACL将匹配并写上这些属性将被授予。

到目前为止，可以与dnattr关键字来获得此相同的行为。随着套，但是，我们可以进一步增强这样的ACL。比方说，我们要允许经理的秘书也更新这些属性。 这是我们的做法：
```
 access to dn.exact="uid=john,ou=people,dc=example,dc=com"
    attrs=carLicense,homePhone,mobile,pager,telephoneNumber
    by self write
    by set="this/manager & user" write
    by set="this/manager/secretary & user" write
    by * read
```
现在，我们需要一张图片，以帮助解释正在发生的事情在这里（缩短了清晰的条目）：

![图8.2：Sets通过项跳转](http://www.openldap.org/doc/admin24/set-following-references.png)

图8.2：Sets通过项跳转

在这个例子中，简是约翰的经理玛丽秘书。这整个关系定义了经理和秘书的属性，二者都使用语法distinguishedName（即完整DNs）。所以，当`uid=john`条目被访问时，this/经理/秘书组变为`{"uid=jane,ou=people,dc=example,dc=com"}`（按照图片中的引用）：
```
 this = [uid=john,ou=people,dc=example,dc=com]
 this/manager = \
   [uid=john,ou=people,dc=example,dc=com]/manager = uid=mary,ou=people,dc=example,dc=com
 this/manager/secretary = \
   [uid=mary,ou=people,dc=example,dc=com]/secretary = uid=jane,ou=people,dc=example,dc=com
```
最终的结果是，当简访问约翰的条目，她将被授予指定属性的写权限。更重要的是，这会发生在她访问的任何条目具有玛丽经理的权限。

这是非常酷和nice，但或许给了秘书太多的权力。也许我们需要进一步限制它。例如，我们只允许执行秘书有这个权力：
```
 access to dn.exact="uid=john,ou=people,dc=example,dc=com"
   attrs=carLicense,homePhone,mobile,pager,telephoneNumber
   by self write
   by set="this/manager & user" write
   by set="this/manager/secretary &
           [cn=executive,ou=group,dc=example,dc=com]/member* &
           user" write
   by * read
```
这几乎和之前的ACL一样，但我们现在还要求连接的用户是`cn=executive`组（可能是嵌套）的成员。

# 9.Limits

# 10.Database Creation and Maintenance Tools

# 11.Backends

# 12.Overlays

# 13. Schema Specification

# 14. Security Considerations

# 15. Using SASL

# 16.使用TLS
OpenLDAP的客户端和服务器能够使用传输层安全（TLS）框架来提供完整性和机密性保护和支持使用SASL扩展机制LDAP身份验证。 TLS在RFC4346中定义。

> 注意：对于产生荣誉证书，请参考http://www.openldap.org/faq/data/cache/185.html

## 1.TLS证书
TLS使用X.509证书进行客户端和服务器的身份。所有的服务器都需要有有效的证书，而客户端证书是可选的。客户端必须要通过SASL外部认证有效证书。有关创建和管理证书的详细信息，请参阅OpenSSL，GnuTLS，或MozNSS文件，这取决于TLS使用的是哪一个库。

### 1.服务器证书
服务器证书的DN必须使用CN属性来命名服务器，并且CN必须携带服务器的完全合格的域名。其他别名和通配符可能存在SubjectAltName证书扩展。对服务器证书名称的详细细节在RFC4513。

### 2.客户端证书
客户端证书的DN，可以直接使用作为认证DN。因为X.509是X.500标准，LDAP的一部分也是基于X.500的，都使用相同的DN格式，一般的DN在用户的X.509证书应该是相同的，以它们的LDAP条目的DN 。但是，有时的DN可能不是完全相同的，因此，在映射认证身份描述的映射设施可以适用于这些的DN。

## 2.TLS配置
获得要求的证书后，有多种选择，必须在客户端和服务器，以使TLS和利用证书都配置。至少，客户端必须包含所有的证书颁发机构（CA）证书将信任的文件名进行配置。服务器必须与CA证书，也是它自己的服务器证书和私钥进行配置。

通常，单个CA将颁发服务器证书和所有受信任的客户端证书，所以服务器只需要信任一个签名CA.但是，一个客户可能希望连接到各种不同组织管理的安全服务器，具有许多不同的CA产生的服务器证书。这样，一个客户可能需要在其配置许多不同的可信CA的列表。

### 1.服务器配置
对于slapd的配置指令属于slapd.conf（5）的全局指令部分。

#### 1.`TLSCACertificateFile <filename>`
该指令指定slapd将信任的包含CA的证书PEM格式文件。对于CA签署服务器证书的证书必须包含这些证书。如果签名CA不是顶层（根）CA，对于CA的自签名CA到顶层的CA整个序列证书应存在。多个证书只是附加到文件;顺序并不重要。

#### 2.`TLSCACertificatePath <path>`
该指令指定包含在单独的文件单个CA证书的目录的路径。此外，该目录必须使用专门在OpenSSL c_rehash工具进行管理。使用此功能时，OpenSSL库将尝试根据自己的名称和序列号的哈希来定位证书文件。该c_rehash实用工具用于生成指向实际证书文件散列名称符号链接。因此，该选项只能与实际支持符号链接文件系统使用。在一般情况下，它是简单的使用TLSCACertificateFile指令代替。

当使用Mozilla NSS，这个指令可以用来指定包含NSS证书和密钥数据库文件的目录的路径。 certutil命令可用于添加CA证书：
```
certutil -d <path> -A -n "name of CA cert" -t CT,, -a -i /path/to/cacertfile.pem
```
此命令将添加一个CA certficate存储在文件名为/path/to/cacertfile.pem的PEM（ASCII）格式的文件中。 -t CT,,意味着证书信任是发行于TLS客户端和服务器使用证书的CA.

#### 3.`TLSCertificateFile <filename>`
该指令指定包含slapd服务器证书的文件。证书一般都是公共信息，不需要特别的保护。

当使用Mozilla NSS，如果使用证书/密钥数据库（含TLSCACertificatePath指定），该指令指定要使用的证书的名称：
```
TLSCertificateFile Server-Cert
```
如果使用的不是建在令牌内部以外的令牌，指定令牌的名头，后面跟一个冒号：
```
TLSCertificateFile my hardware device:Server-Cert
```
使用的certutil-L列出证书的名字：
```
certutil -d /path/to/certdbdir -L
```

#### 4.`TLSCertificateKeyFile <filename>`
该指令指定包含存储在TLSCertificateFile文件中的证书相匹配的私有密钥的文件。私钥本身是敏感数据，通常是密码保护的加密。不过，目前的实现不支持加密的密钥所以关键不得加密，文件本身必须小心保护。

当使用Mozilla NSS，这个指令指定包含与TLSCertificateFile指定的证书密钥的密码的文件的名称。该modutil工具命令可用于关闭密码保护的证书/密钥数据库。例如，如果TLSCACertificatePath其中指定的/ etc / openldap中/ certdb作为证书/密钥数据库的位置，使用modutil工具将密码更改为空字符串：
```
modutil -dbdir /etc/openldap/certdb -changepw 'NSS Certificate DB'
```
你必须有旧密码，如果有的话。忽略有关正在运行的警告浏览器。按下“Enter”新密码。

#### 5.`TLSCipherSuite <cipher-suite-spec>`
此指令配置的什么密码将被接受和接受优先顺序。`<cipher-suite-spec>`应该是OpenSSL的密码规范。您可以使用命令`openssl ciphers -v ALL`获得可用的密码规范的详细列表。

除了个人密码的名字，HIGH,MEDIUM,LOW,EXPORT,和EXPORT40可能会有所帮助，使用TLSv1，SSLv3，以及SSLv2。
要获得的GnuTLS密码的列表使用命令：`gnutls-cli -l`

当使用Mozilla NSS，OpenSSL的密码套件规范使用并转换成由Mozilla NSS内部使用的格式。没有一个简单的方法来列出命令行的密码套件。权威列表是的Mozilla NSS在文件sslinfo.c在结构的源代码
```
static const SSLCipherSuiteInfo suiteInfo[]
```

#### 6.`TLSRandFile <filename>`

这个指令指定的文件，以获得时的/ dev / urandom设备无法使用随机位。如果该系统提供的/ dev / urandom的则不需要此选项，否则随机数据源必须配置。有些系统（如Linux）的默认提供的/ dev / urandom的，而其他人（例如Solaris）必须有一个补丁的安装提供了，和其他人可能不支持它。在后一种情况下，EGD或PRNGD应安装，以及该指令应指定EGD/ PRNGD插座的名称。环境变量RANDFILE也可用于指定的文件名。另外，在不存在这些选项中，可以如果存在用于slapd的用户的主目录中的.rnd文件。要使用.rnd文件，只需创建该文件，任意数据的几百个字节复制到文件中。该文件仅用于提供伪随机数发生器的种子，并且它不需要非常多的数据来工作。

该指令用的GnuTLS和Mozilla NSS忽略。

#### 7.`TLSEphemeralDHParamFile <filename>`
该指令指定包含的Diffie-Hellman临时密钥交换参数的文件。这是必需的，以使用一个DSA证书上的服务器侧（即TLSCertificateKeyFile点DSA密钥）。可被包括在该文件中的参数多套;所有的人都将被处理。参数可以使用下面的命令来生成
```
openssl dhparam [-dsaparam] -out <filename> <numbits>
```
该指令忽略GnuTLS和Mozilla NSS。

#### 8.`TLSVerifyClient { never | allow | try | demand }`

该指令规定了什么会检查客户端证书在传入TLS会话，如果有演出。这个选项被设置为永不默认，在这种情况下，服务器从未询问客户端证书。随着允许服务器将要求客户端证书的设置;如果没有正常提供会话继续。如果提供了证书，但该服务器无法验证，证书将被忽略，会话正常进行，就好像已经提供过任何证书。随着尝试证书的设置要求，如果没有提供，会话继续正常进行。如果提供的证书，它不能被验证，则会话立即终止。随着需求的设置证书，并要求必须提供有效的证书，否则会话立即终止。

> 注：服务器必须按顺序使用SASL外部认证机制与TLS会话请求客户端证书。因此，非默认设置TLSVerifyClient必须配置之前SASL外部认证可以尝试，而SASL EXTERNAL机制将只提供给客户，如果收到一个有效的客户端证书。

### 2.客户端配置
大多数客户端配置指令并行的服务器指令。该指令的名称是不同的，并且它们进入的ldap.conf（5）代替slapd.conf中（5），但是它们的功能是大多相同的。此外，虽然大多数的这些选项可以在系统范围的基础上进行配置，它们可以全部由在其.ldaprc文件个别用户重写。

该LDAP启动TLS操作在LDAP用于启动TLS协商。所有的OpenLDAP命令行工具支持-Z和-zz标志来指示启动TLS操作是否发行。后者标志指示工具是停止处理，如果前者允许命令继续TLS不能启动。

在LDAPv2的环境中，TLS使用LDAP安全URI方案正常启动（LDAPS：//）代替正常的LDAP URI方案（LDAP：//）。 OpenLDAP的命令行工具可以让任何计划与-H标志，并与URI的ldap.conf（5）选项一起使用。

#### 1.`TLS_CACERT <filename>`
这等同于服务器的TLSCACertificateFile选项。如在TLS配置部分所指出的，客户端通常可能需要了解比服务器多个CA，但在其他方面相同的考虑也适用。

#### 2.`TLS_CACERTDIR <path>`
这等同于服务器的TLSCACertificatePath选项。指定的目录必须与OpenSSL的c_rehash工具以及进行管理。如果使用Mozilla NSS，`<path>`可能包含一个证书/密钥数据库。

#### 3.`TLS_CERT <filename>`

该指令指定包含客户端证书的文件。这是一个用户只指令，并且只能在用户的.ldaprc文件中指定。

当使用Mozilla NSS，如果使用证书/密钥数据库（含TLS_CACERTDIR指定），该指令指定要使用的证书的名称：
```
TLS_CERT Certificate for Sam Carter
```
如果使用的不是建在令牌内部以外的令牌，指定
令牌的名头，后面跟一个冒号：
```
TLS_CERT my hardware device:Certificate for Sam Carter
```
使用的certutil-L列出名字的证书：
```
certutil -d /path/to/certdbdir -L
```

#### 4.`TLS_KEY <filename>`
该指令指定包含存储在文件TLS_CERT证书相匹配的私有密钥的文件。对于TLSCertificateKeyFile提到的相同的限制适用于此。这也是一个用户只指令。

#### 5.`TLS_RANDFILE <filename>`
这个指令是一样的服务器的TLSRandFile选项。

#### 6.`TLS_REQCERT { never | allow | try | demand }`
该指令等同于服务器的TLSVerifyClient选项。然而，对于客户端的默认值是需求，一般不存在充分的理由更改此设置。

# 17.构建分布式目录服务

# 18.复制
为了提供一个有弹性的企业部署，复制目录是一个基础需求.

OpenLDAP有多种配置选项来建立一个可复制的目录. 在前一个版本里面, 复制被限定在一个主服务器和若干个从服务器的条件下来讨论。一个主服务器从其他客户端接受目录更新, 而一个从服务器则仅仅从一个（单个的）主服务器接受更新. 这个复制结构被僵化地定义并且任何典型的数据库只能完成一个单一角色,主或者从.

现在OpenLDAP支持一个更广泛的复制拓扑, 关于提供者和消费者的以下这些条件已经不推荐了: 一个提供者复制目录更新到消费者; 消费者从提供者接收复制更新. 不像僵化定义的主/从关系,提供者/消费者角色更加的流动化：一个接收复制更新的消费者可能传递给其它服务器的另一个消费者，所以一个消费者也可以同时成为一个提供者。而且，消费者不需要成为一个实际上的LDAP服务器;它也可以仅仅是一个LDAP客户端。

以下章节将描述复制技术和讨论各种可用的复制选项. 

## 1.复制技术
### 1.LDAP同步复制
LDAP同步复制引擎, 简称syncrepl, 是一个消费方的复制引擎，能让消费者服务器维护一个抽取片断的影子副本. 一个syncrepl引擎以slapd的一个线程的方式驻留在消费者那里. 它建立和维护一个消费者复制，方法是连接一个复制提供者去执行初始化DIT内容载荷以及接下来的定期的内容拉取或及时根据内容变更来更新。

Syncrepl 使用LDAP内容同步协议(或简称 LDAP Sync) 作为复制同步协议. LDAP Sync 提供一个有状态的复制，它同时支持拉模式和推模式同步并且不要求使用历史存储. 在拉模式复制下消费者定期拉提供者服务器的内容来更新. 在推模式复制下消费者监听提供者实时发送的更新. 因为协议不要求历史存储, 提供者不需要维护任何它接收到的更新的日志. (注意syncrepl引擎是可扩展的，并支持未来新增的复制协议.)

Syncrepl通过维护和交换同步cookies来保持对复制内容的状态的跟踪. 因为syncrepl消费者和提供者维护它们的内容状态, 消费者可以拉取提供者的内容来执行增量同步，只要请求那些最新的提供者内容条目。 Syncrepl也通过维护复制状态方便了复制的管理. 消费者复制可以在任何同步状态下从一个消费方或一个提供方的备份来构建. Syncrepl能自动重新同步消费者复制到和当前的提供者内容一致的最新状态.

Syncrepl同时支持拉模式和推模式同步. 在它的基本的 refreshOnly 同步模式下, 提供者使用基于拉模式的同步,这里消费者服务器不需要被跟踪并且不维护历史信息. 需要提供者处理的定期的拉请求信息，包含在请求本身的同步cookie里面。为了优化基于拉模式的同步, syncrepl把LDAP同步协议的当前阶段当成它的删除阶段一样处理, 而不是频繁地回滚完全重载. 为了更好地优化基于拉模式的同步, 提供者可以维护一个按范围划分的会话日志作为历史存储. 在它的 refreshAndPersist 同步模式, 提供者使用基于推模式的同步. 提供者维护对请求了一个持久性搜索的消费者服务器的跟踪，并且当提供者复制内容修改的时候向它们发送必要的更新.

有了syncrepl, 如果消费者服务器有对被复制的DIT片断的适当的操作权限，一个消费者服务器可以建立一个复制而不修改提供者的配置并且不需要重新启动提供者服务器. 消费者服务器可以停止复制，也不需要提供方的任何变更和重启.

Syncrepl支持局部的，稀疏的和片断复制. 影子DIT片断由一个标准通用搜索来定义，包括基础，范围，过滤条件，和属性列表. 复制内容也受限于syncrepl复制连接的绑定用户的操作权限. 

#### 1.LDAP内容同步协议
LDAP同步协议允许一个客户端维护DIT片段的一个同步副本。LDAP同步操作的定义是一套控制,以及扩展LDAP搜索操作的其它协议元素。本节仅简单介绍LDAP内容同步协议。欲了解更多信息，请参阅RFC4533 。

LDAP同步协议支持拉和监听变更，通过定义两个各不相同的同步操作： refreshOnly和refreshAndPersist 。拉是由refreshOnly操作实现的。消费者使用一个LDAP搜索请求拉提供者，附带LDAP同步控制。消费者副本通过使用传回搜索到的信息实现拉取， 来从供应商副本同步。提供者在正常搜索时，搜索行动结束返回SearchResultDone，来表示完成搜索。监听是由refreshAndPersist操作实施的。顾名思义，它首先是一个搜索，如refreshOnly 。不是在返回所有目前搜索条件相匹配的条目后就完成搜索，而是同步搜索仍然保持供应商的持久化。随后供应商的同步内容更新产生额外的条目更新发送给消费者。

refreshOnly操作和refreshAndPersist操作的刷新阶段，可以在当前阶段或删除阶段执行。

在当前阶段，提供者发送自上次同步以来搜索范围内更新的条目给消费者。提供者发送更新的条目的所有请求的属性，不论它们改变与否。对于每个仍然保持不变范围的条目，提供者发送一个当前的消息，只包含条目的名称和代表"当前"状态的同步控制。当前消息不包含条目的任何属性。在消费者收到所有更新和当前条目之后，它能够可靠地确定新的消费者副本，通过增加那些条目供应商新增的条目，和替换那些供应商修改的条目，并删除消费副本中没有更新过也没有被提供者指定为"当前"的条目。

更新条目的传播在删除阶段和在当前阶段是一样的。提供者发送搜索范围内自上次同步以来更新的条目的所有请求的属性，给消费者。在删除阶段，无论如何，供应商为搜索范围内每个删除的条目发送一个删除消息，而不是发送当前消息。删除消息只包含条目的名称代表"删除"状态和同步控制。新的消费副本的决定可根据附加在SearchResultEntry的同步控制信息来增加，修改和删除条目。

如果LDAP同步提供者维护了一个历史存储，而且可以确定哪些条目的范围超出了自上次同步时间以来的消费者副本，供应商可以使用删除阶段。如果供应商不保持任何记录存储，无法从历史存储确定范围之外的条目，或存储的历史并不包括消费者的过时的同步状态，供应商应利用当前阶段。比起全部内容重载产生的同步通信来，使用当前阶段是更有效的。为了更好的减少同步他通讯量，LDAP同步协议还提供了一些优化，如标准化entryUUIDs的传输和一个单一syncIdSet讯息中传送多种entryUUIDs 。

在refreshOnly同步的结尾，在同步完成之后，提供者发送一个同步cookie到消费者，作为消费者副本的一个状态指标。当消费者向供应商请求下次的增量同步时，它将展示收到的Cookie。

当使用 refreshAndPersist同步时，提供者在刷新阶段的结尾发送一个同步Cookie，通过发送一个同步信息消息refreshDone =为TRUE 。它还发送一个同步的Cookie附加到同步搜索的持久化阶段产生的SearchResultEntry消息中。在持久化阶段，提供者还可以发送一个同步信息，包含同步的Cookie，在任何提供者要更新消费端状态指标的时候。

在LDAP同步协议，条目具有唯一的entryUUID属性值。它可以作为条目的一个可靠的标识符。条目的DN，另一方面，可随时间变化，因此不能被视为可靠的标识符。entryUUID附在每个SearchResultEntry或SearchResultReference上作为同步控制的一个部分。 

#### 2.Syncrepl细节
syncrepl引擎同时使用LDAP同步协议的refreshOnly和refreshAndPersist操作. 如果一个syncrepl规范存在于一个数据库定义中, slapd(8) 以一个 slapd(8) 线程的方式启动一个syncrepl引擎并规划它的执行时间表. 如果指定了refreshOnly操作, syncrepl引擎在一个同步操作完成之后将按间隔时间重新排程. 如果指定了refreshAndPersist操作, 引擎将保持激活并从提供者服务器处理持久性同步消息.

syncrepl引擎同时应用刷新同步的当前阶段和删除阶段. 可以在提供者服务器配置一个会话日志存储一定数量的从数据库中删除的entryUUIDs。多复制共享相同的会话日志. 如果会话日志是当前的并且消费者服务器足够新以至于在客户端的最后一次同步之后没有会话日志条目被删除，那么syncrepl引擎使用删除阶段. 如果没有为复制内容配置会话日志或如果消费者复制太陈旧而无法被会话日志涵盖到, syncrepl引擎使用当前阶段. 目前会话日志存储的设计是基于内存的, 所以包含在会话日志的信息相对多提供者的调用不是持久性的. 目前它不支持通过使用LDAP操作来操作会话日志存储. 它目前也不支持对会话日志施加访问控制.

作为进一步的优化, 甚至同步搜索都不和任何会话日志关联, 当没有发生复制相关的更新时将不会有任何条目传输给消费者.

syncrepl引擎, 是一个消费方的复制引擎, 可以工作在任何后端. LDAP同步提供者可以在任何后端配置成一个 overlay , 但是最好工作在 back-bdb 或 back-hdb 后端.

LDAP同步提供者为每一个数据库维护一个 contextCSN 作为提供者内容的当前同步状态指标. 它是提供者范围的最大 entryCSN，所以对于更小的拥有悬而未决的entryCSN值的条目来说不存在事务. contextCSN不能只是设成最大的已发表的entryCSN，因为 entryCSN 是在一个事务开始之前获得的并且事务还未提交到发表序列.

提供者在context suffix 条目的 contextCSN 属性存储 上下文的contextCSN . 这个属性不是在每个更新操作之后写入数据库; 而是主要在内存中维护. 在数据库启动时间提供者读取最后一次存储的 contextCSN 到内存里并且此后就只使用内存内的拷贝. 缺省的, 对 contextCSN 的变更作为一个数据库更新的结果将不写入数据库，直到服务器完全干净地关机. 如果需要的话，设置一个检查点可以让contextCSN写出得更频繁一些.

注意在启动的时间, 如果提供者不能从suffix条目读取一个 contextCSN , 它将扫描整个数据库来决定它的值, 并且在一个大的数据库中扫描可能要花很长时间. 当一个 contextCSN 值被读取, 这个数据库将仍被扫描用于任何高于它的 entryCSN 值, 以确保 contextCSN 值真的反应了数据库中entryCSN的最大提交 . 在支持不等式索引的数据库中, 在 entryCSN 属性上设置一个 eq 索引并配置 contextCSN 检查点，将极大地加速这个扫描步骤.

如果通过读取和扫描数据库没有决定 contextCSN, 一个新的值将被生成. 而且, 如果扫描数据库产生了一个比之前纪录在suffix条目中的contextCSN属性更大的entryCSN，一个检查点将立刻写入新的值.

消费者也存储它的复制状态, 它是作为一个同步cookie接收的提供者的contextCSN, 在suffix条目的contextCSN属性. 当它执行对提供者服务器的顺序增量同步时,由一个消费者服务器维护的复制状态被用作同步状态指标. 当它在一个级联复制配置中承当一个第二提供者服务器时,它也被用作提供方的同步状态指标. 因为消费者和提供者状态信息是在它们各自的服务器的同一个地方维护的, 任何消费者可以被提拔成为提供者(反之亦然)而不需要任何特别的动作.

因为在syncrepl规范中可能使用一个通用搜索过滤器, 上下文中的一些条目可能被从同步内容中省略了. syncrepl引擎建立一个粘条目来填充复制上下文中的窟窿，如果复制内容的任何部分属于这个窟窿的话。 这些粘条目在搜索结果中将不返回，除非提供了ManageDsaIT控制。

另外，作为在syncrepl规范使用搜索过滤器的结果, 可能会有类似这样的修改，即从复制范围移除一个条目，即使提供者上的条目还没有被删除。逻辑上这个条目必须在消费者服务器被删除但是在refreshOnly模式下，如果没有会话日志则提供者无法检测和传播这个变更.

关于配置，参见 Syncrepl 节. 

## 2.部署替代
LDAP同步协议只对复制规定了狭窄的范围,OpenLDAP实现则是极为弹性的并且支持各种操作模式以处理协议中未显式地提出的其他情景. 

### 1.Delta-syncrepl复制
> * LDAP同步复制的缺点: 

LDAP同步复制是一个基于对象的复制机制. 当提供者的一个被复制对象中的任何属性值改变时, 每个消费者在复制过程中撷取并处理完整的变更对象, 包括所有改变和没改变的属性值. 这方法的一个好处是当多个变更发生在单一对象上时, 那些变更的精确顺序不需要保存; 只有最终状态是有意义的. 但是当使用模式(匹配的方式)在一次变更中处理很多对象时，这个方法可能有缺点。

例如, 假设你有一个数据库包含 100,000 对象，每个对象是 1 KB . 进一步, 假设你经常运行一个批处理工作来变更主服务器上的 100,000 对象的每一个对象中的一个两字节的属性值. 不算LDAP和TCP/IP协议的开销, 每次你运行这个工作每个消费者将传送并处理 1 GB 的数据，只是为了处理这个 200KB 的变更!

在类似这样的案例中，99.98% 被传送和处理的数据将是多余的, 因为它们代表那些未变更的值. 这是一个对宝贵的传输和处理带宽的浪费并且可能导致发展出不可接受的复制日志的积压. 虽然这个情形是一个极端, 但它有助于演示某些LDAP部署的一个非常真实的问题. 

> * 看看Delta-syncrepl怎么处理: 

Delta-syncrepl, 一个基于变更日志syncrepl变种, 被设计用来处理类似上面所说的情况. Delta-syncrepl通过在提供者一端维护一个可选择深度的变更日志来起作用. 复制消费者为它需要的变更检查这个变更日志，只要变更日志包含它需要的变更，消费者就从变更日志撷取这些变更并把它们应用到自己的数据库. 不过，一个复制（译者注：指变更日志里的变更）如果离上一次同步的状态太远(或消费者根本就是空的), 可以用常规的syncrepl把它（指消费者）恢复到最新的状态然后复制重新切换到delta-syncrepl模式.

关于配置请参考 Delta-syncrepl 章节. 

### 2.N-Way Multi-Master复制
Multi-Master复制是一个使用Syncrepl复制数据到多个提供者（“主服务器”）目录服务器的复制技术. 

#### 1.对于Multi-Master replication有效的观点
> * 如果任何提供者失败了, 其他提供者将继续接受更新 
> * 避免了单点失败 
> * 提供者们可以在不同的物理位置例如跨越全球网络. 
> * 好的自动容错/高可用性 

#### 2.对于Multi-Master replication无效的观点
(这些经常被声称是Multi-Master复制的优点但是那些说法是错误的): 
> * 它不关负载均衡任何事  
> * 提供者必须对所有其他的服务器进行写操作，这意味着分布在所有的服务器上的网络交通和写操作负载,和单一主服务器是一样的。 
> * 多服务器的服务器利用率和负载在最好的情况下和单服务器一样; 最坏的情况下单服务器更优，因为在提供者和消费者之间使用不同的模式的时候索引可以做出不同的优化调整. 

#### 3.和Multi-Master replication抵触的观点
> * 打破了目录模式的数据一致性的保障 
> * http://www.openldap.org/faq/data/cache/1240.html
> * 如果提供者的连接因为网络问题丢失了, 那么 "自动容错" 只会使问题复杂化 
> * 通常, 一个特定的机器不能区分失去和一个节点的联系是因为该节点崩溃了还是因为网络连接失败了a 
> * 如果一个网络是分割开的而多个客户端开始向每一个"主服务器"写操作，那么和解将是一个痛苦; 可能最好的办法是禁止那些被单一提供者分隔开的客户端的写操作 

关于配置，请看下面的 N-Way Multi-Master 章节。

### 3.MirrorMode复制
MirrorMode是一个混合配置，既提供单主服务器复制的所有一致性保障，也提供多主服务器模式的高可用性. 在 MirrorMode 两个提供者都被设置成从对方复制(就象一个多主服务器配置), 但是一个额外的前段被用来引导所有的写操作到仅仅到两台服务器中的其中一台. 第二个提供者将只在第一台服务器崩溃时进行写操作, 那时这个前端将切换路径引导所有的写操作到第二个提供者. 当一个崩溃的提供者被修复并且重启动后将自动从正在运行的提供者那里活得任何更新并重新同步. 

#### 1.MirrorMode的观点
> * 对于目录的写操作提供了一个高可用性 (HA) 方案(复制处理读操作)  
> * 只有一个提供者是可操作的l, 写操作的安全是可接受的 
> * 提供者节点从对方互相复制, 所以它们总是最新的并且可以随时准备好接管 (热备份) 
> * Syncrepl也允许提供者节点在任何停机时间进行重新同步 

#### 2.和MirrorMode抵触的观点
> * MirrorMode 不能被称为多主机方案.这是因为同一时间写操作不得不仅限于镜像节点中的一个
> * MirrorMode 可被称为Active-Active Hot-Standby（“双活热备份”,呃，这个翻译怎么样，传神不？）, 因此需要一个额外的服务器(代理模式的slapd)或设备(硬件负载平衡装置)来管理哪个提供者是当前激活的  
> * 备份的管理稍微不同
>     * 如果备份bdb本身并且定期备份事务日志文件，那么镜像对的相同数字需要用于收集日志文件直到下一次数据库备份发生 

### 4.Syncrepl代理模式
因为LDAP同步协议同时支持基于“拉”和“推”的复制, “推”模式 (refreshAndPersist) 在提供者开始"推"变更之前仍必须由消费者初始化. 在一些网络配置中, 特别是防火墙限制了连接的方向时, 一个提供者初始化的推模式是需要的.

这个模式可以被配置成LDAP Backend (Backends and slapd-ldap(8)). 不用在实际的消费者服务器上运行syncrepl引擎, 而是一个slapd-ldap代理设置在靠近（或搭配在）提供者的地方指向消费者, 而这个syncrepl引擎运行在这个代理服务器上.

关于配置, 请看 Syncrepl代理 章节. 

#### 1.替代Slurpd
旧的slurpd机制只操作主服务器初始化的推模式. Slurpd复制被Syncrepl复制取代了并且在OpenLDAP 2.4中被完全移除了.

slurpd守护进程是原来继承自UMich's LDAP的复制机制并且以推模式操作: 主服务器推变更到从服务器. 因为多种原因它被替换掉, 简短的说: 
> * 它是不可靠的
>     * 它对replog中的记录的次序极为敏感
>     * 它可能很容易失去同步, 这时需要手工干预来从主目录重新同步从服务器数据库
>     * 它对不可用的服务器不是非常宽容. 如果一个从服务器长时间停机, replog可能变得太大以至于slurpd无法处理 
> * 它只工作在推模式
> * 它需要停止和重新启动主服务器来增加从服务器 
> * 它只支持单一主服务器复制 

Syncrepl没有那些弱点: 
> * Syncrepl是自同步的; 你可以在任何状态启动一个消费者数据库，从完全空的到完全同步的，它将自动做正确的事来完成和维护同步
>     * 它对变更发生的次序完全不敏感 
>     * 它保障消费者和提供者内容的合流,不用手工干预 
>     * 无论一个消费者多长时间没有联系提供者，它都能重新同步 
> * Syncrepl能双向操作 
> * 消费者能在不用碰提供者的情况下被加入 
> * 支持多主服务器复制 

## 3.配置不同的复制类型
### 1.Syncrepl
#### 1.Syncrepl配置
因为syncrepl是一个消费方的复制引擎, syncrepl规范定义在 slapd.conf(5) 的消费者服务器, 而不是在提供者的服务器配置文件里. 复制内容的初始化装载可以有两种执行方式，以无同步cookie的方式启动一个syncrepl 引擎，或装载一个提供者服务器的全备份LDIF文件填充到消费者服务器.

当从一个备份装载的时候, 它不需要执行从提供者内容的最新备份初始化装载这个动作. syncrepl引擎将自动同步初始化的消费者复制当前的提供者内容. 结果是, 它不需要为了避免由于内容备份和装载过程中提供者服务器仍在更新而导致复制不一致的问题来停止提供者服务器.

当复制一个大规模的目录时, 特别是在一个带宽受限的环境, 建议从备份装载消费者而不是使用syncrepl执行一个完全的初始化装载. 

#### 2.设置提供者的slapd
提供者被实现为一个 overlay, 所以这个 overlay 本身在使用之前必须首先如 slapd.conf(5) 配置. 提供者只有两个配置指示, 在 contextCSN 上设定检查点和配置会话日志. 因为 LDAP 同步搜索受限于访问控制, 应为复制的内容设置正确的访问控制权限.

contextCSN检查点设置如下 
```
syncprov-checkpoint <ops> <minutes>
```
检查点只在成功的写操作之后测试. 如果 <ops> 操作了或从上次检查点到现在超过了 <minutes> 时间, 将执行一个新的检查点.

会话日志设置如下 
```
syncprov-sessionlog <size>
```
这里 <size> 是会话日志可以记录的条目的最大数量. 当一个会话日志被配置好, 它就自动用于所有对此数据库的 LDAP 同步搜索.

注意使用会话日志需要搜索 entryUUID 属性. 在这个属性上设一个 eq 索引将极有益于提供者服务器的会话日志的性能.

slapd.conf(5)中一个更复杂的例子内容如下: 
```
database bdb
suffix dc=Example,dc=com
rootdn dc=Example,dc=com
directory /var/ldap/db
index objectclass,entryCSN,entryUUID eq

overlay syncprov
syncprov-checkpoint 100 10
syncprov-sessionlog 100
```
#### 3.设置消费者的slapd
在 slapd.conf(5) 的replica范围的数据库一节定义了syncrepl复制.syncrepl引擎是独立的后端并且可以使用任何数据库类型定义directive.
```
database hdb
suffix dc=Example,dc=com
rootdn dc=Example,dc=com
directory /var/ldap/db
index objectclass,entryCSN,entryUUID eq

syncrepl rid=123
    provider=ldap://provider.example.com:389
    type=refreshOnly
    interval=01:00:00:00
    searchbase="dc=example,dc=com"
    filter="(objectClass=organizationalPerson)"
    scope=sub
    attrs="cn,sn,ou,telephoneNumber,title,l"
    schemachecking=off
    bindmethod=simple
    binddn="cn=syncuser,dc=example,dc=com"
    credentials=secret
```
在这个例子中, 消费者将从ldap://provider.example.com的389端口连接到提供者 slapd(8) 来执行每天一次同步的拉操作(refreshOnly)模式. 它将以 cn=syncuser,dc=example,dc=com 绑定，以密码"secret"进行简单验证. 注意要在提供者服务器为cn=syncuser,dc=example,dc=com设置适当的访问控制权限以接收想要的复制内容. 另外提供者上的搜索限制必须足够高以允许同步用户接收请求内容完整的拷贝. 消费者使用 rootdn 写入它的数据库所以它总是有全部的权限来写所有的内容.

在上面的例子中同步搜索将在dc=example,dc=com的整个子树搜索 objectClass 是 organizationalPerson 的条目. 请求的属性是 cn, sn, ou, telephoneNumber, title, 和 l. schema 检查被关闭，这样当处理从提供者slapd(8)来的更新时消费者 slapd(8) 将不会强制对条目进行 schema 检查.

更多的详细信息参见 syncrepl 指示, 见本管理指南的slapd配置文件的syncrepl节. 

#### 4.启动提供者和消费者的slapd
提供者slapd(8)不需要重启. contextCSN将会根据需要自动生成: 它可能原来就包含在 LDIF 文件里, 由 slapadd (8) 生成, 在上下文中通过变更生成, 或当第一次 LDAP 同步搜索到达提供者时生成. 如果装载了一个之前不包含contextCSN的LDIF 文件, slapadd (8) 应使用 -w 选项来令它生成. 这将使服务器第一次运行时变得快一点.

当启动一个消费者 slapd(8) 时, 为了从一个特定的状态开始同步，它可能使用命令行参数 -c 即cookie选项，以提供一个同步cookie. cookie是一个逗号分隔的name=value对的列表. 目前支持的 syncrepl cookie 字段是 csn=<csn> 和 rid=<rid>. <csn>代表消费者复制的当前同步状态. <rid> 标识这个消费者服务器的一个本地消费者复制. 它用于把cookie关联到slapd.conf(5)中拥有匹配的复制标识的 syncrepl 定义. <rid>必须超过三位数. 命令行cookie会覆盖存储在消费者复制数据库中的同步cookie. 

### 2.Delta-syncrepl
#### 1.Delta-syncrepl提供者配置
设置 delta-syncrepl 需要同时改变主服务器和复制服务器的配置: 
```
     # 给予复制DN无限的读权限.  这个 ACL 需要和其他
     # ACL 声明合并, 并且/或者在数据库范围内移动
     # "by * break" 部分会执行随后的规则
     # 细节请看 slapd.access(5) .
     access to *
        by dn.base="cn=replicator,dc=symas,dc=com" read
        by * break
 
     # 设置模块路径
     modulepath /opt/symas/lib/openldap
 
     # 装载 hdb 后端
     moduleload back_hdb.la
 
     # 装载操作日志 overlay
     moduleload accesslog.la
 
     #装载 syncprov overlay
     moduleload syncprov.la
 
     # 操作日志数据库定义
     database hdb
     suffix cn=accesslog
     directory /db/accesslog
     rootdn cn=accesslog
     index default eq
     index entryCSN,objectClass,reqEnd,reqResult,reqStart
 
     overlay syncprov
     syncprov-nopresent TRUE
     syncprov-reloadhint TRUE
 
     # 让复制 DN 有无限的搜索权限
     limits dn.exact="cn=replicator,dc=symas,dc=com" time.soft=unlimited time.hard=unlimited size.soft=unlimited size.hard=unlimited
 
     # 主数据库定义
     database hdb
     suffix "dc=symas,dc=com"
     rootdn "cn=manager,dc=symas,dc=com"
 
     ## 任何期望的其他配置选项
 
     # syncprov 特别索引
     index entryCSN eq
     index entryUUID eq
 
     # 主数据库的syncrepl提供者
     overlay syncprov
     syncprov-checkpoint 1000 60
 
     # 主数据库的操作日志overlay定义
     overlay accesslog
     logdb cn=accesslog
     logops writes
     logsuccess TRUE
     # 每天扫描一次操作日志数据库, 并清除7天前的条目
     logpurge 07+00:00 01+00:00
 
     # 让复制DN有无限搜索权限
     limits dn.exact="cn=replicator,dc=symas,dc=com" time.soft=unlimited time.hard=unlimited size.soft=unlimited size.hard=unlimited
```
更多信息, 访问(slapo-accesslog(5) 和 slapd.conf(5))相关的 man 页 

#### 2.Delta-syncrepl消费者配置
```
     # 复制数据库配置
     database hdb
     suffix "dc=symas,dc=com"
     rootdn "cn=manager,dc=symas,dc=com"
 
     ## 任何关于复制的其他配置, 例如你期望的索引
     ## 
 
     # syncrepl特有的索引
     index entryUUID eq
 
     # syncrepl参数
     syncrepl  rid=0
               provider=ldap://ldapmaster.symas.com:389
               bindmethod=simple
               binddn="cn=replicator,dc=symas,dc=com"
               credentials=secret
               searchbase="dc=symas,dc=com"
               logbase="cn=accesslog"
               logfilter="(&(objectClass=auditWriteObject)(reqResult=0))"
               schemachecking=on
               type=refreshAndPersist
               retry="60 +"
               syncdata=accesslog
 
     # 提交更新到主服务器
     updateref               ldap://ldapmaster.symas.com
```
以上配置假定你在你用于绑定到提供者的数据库中有一个复制者标识. 另外, 所有数据库 (主数据库, 复制数据库, 以及操作日志存储数据库) 也应该正确调整 DB_CONFIG 文件以满足你的需要. 

### 3.N-Way Multi-Master
以下例子将使用三个主节点. Keeping in line with test050-syncrepl-multimaster of the OpenLDAP test suite, 我们将通过cn=config配置slapd(8)

这里设置配置数据库: 
```
     dn: cn=config
     objectClass: olcGlobal
     cn: config
     olcServerID: 1
 
     dn: olcDatabase={0}config,cn=config
     objectClass: olcDatabaseConfig
     olcDatabase: {0}config
     olcRootPW: secret
```
第二和第三服务器明显会有一个不同的 olcServerID: 
```
     dn: cn=config
     objectClass: olcGlobal
     cn: config
     olcServerID: 2
 
     dn: olcDatabase={0}config,cn=config
     objectClass: olcDatabaseConfig
     olcDatabase: {0}config
     olcRootPW: secret
```
这里设置 syncrepl 为提供者 (因为这些都是主服务器): 
```
     dn: cn=module,cn=config
     objectClass: olcModuleList
     cn: module
     olcModulePath: /usr/local/libexec/openldap
     olcModuleLoad: syncprov.la
```
现在我们设置第一个主节点 (使用你自己的确切的urls替换掉 $URI1, $URI2 和 $URI3 等.): 
```
     dn: cn=config
     changetype: modify
     replace: olcServerID
     olcServerID: 1 $URI1
     olcServerID: 2 $URI2
     olcServerID: 3 $URI3
 
     dn: olcOverlay=syncprov,olcDatabase={0}config,cn=config
     changetype: add
     objectClass: olcOverlayConfig
     objectClass: olcSyncProvConfig
     olcOverlay: syncprov
 
     dn: olcDatabase={0}config,cn=config
     changetype: modify
     add: olcSyncRepl
     olcSyncRepl: rid=001 provider=$URI1 binddn="cn=config" bindmethod=simple
       credentials=secret searchbase="cn=config" type=refreshAndPersist
       retry="5 5 300 5" timeout=1
     olcSyncRepl: rid=002 provider=$URI2 binddn="cn=config" bindmethod=simple
       credentials=secret searchbase="cn=config" type=refreshAndPersist
       retry="5 5 300 5" timeout=1
     olcSyncRepl: rid=003 provider=$URI3 binddn="cn=config" bindmethod=simple
       credentials=secret searchbase="cn=config" type=refreshAndPersist
       retry="5 5 300 5" timeout=1
     -
     add: olcMirrorMode
     olcMirrorMode: TRUE
```
现在启动主服务器和一个或多个消费者服务器, 也把上面的 LDIF 加入到第一个消费者, 第二个消费者等等. 然后它将复制cn=config. 你现在就在config数据库上拥有了多路多主机.

我们仍不得不复制实际的数据, 而不仅是 config, 所以添加下面这些到主服务器(所有激活的和配置好的消费者/主服务器将领取这个配置, 因为他们都是在同步的). 同样的, 以任何对你的安装可用的设置替换所有 ${} 变量: 
```
     dn: olcDatabase={1}$BACKEND,cn=config
     objectClass: olcDatabaseConfig
     objectClass: olc${BACKEND}Config
     olcDatabase: {1}$BACKEND
     olcSuffix: $BASEDN
     olcDbDirectory: ./db
     olcRootDN: $MANAGERDN
     olcRootPW: $PASSWD
     olcLimits: dn.exact="$MANAGERDN" time.soft=unlimited time.hard=unlimited size.soft=unlimited size.hard=unlimited
     olcSyncRepl: rid=004 provider=$URI1 binddn="$MANAGERDN" bindmethod=simple
       credentials=$PASSWD searchbase="$BASEDN" type=refreshOnly
       interval=00:00:00:10 retry="5 5 300 5" timeout=1
     olcSyncRepl: rid=005 provider=$URI2 binddn="$MANAGERDN" bindmethod=simple
       credentials=$PASSWD searchbase="$BASEDN" type=refreshOnly
       interval=00:00:00:10 retry="5 5 300 5" timeout=1
     olcSyncRepl: rid=006 provider=$URI3 binddn="$MANAGERDN" bindmethod=simple
       credentials=$PASSWD searchbase="$BASEDN" type=refreshOnly
       interval=00:00:00:10 retry="5 5 300 5" timeout=1
     olcMirrorMode: TRUE
 
     dn: olcOverlay=syncprov,olcDatabase={1}${BACKEND},cn=config
     changetype: add
     objectClass: olcOverlayConfig
     objectClass: olcSyncProvConfig
     olcOverlay: syncprov
```

> 注意: 你的所有服务器始终必须使用例如 NTP http://www.ntp.org/, 原子钟, 或一些其他可用的时间参照物紧紧同步.
> 
> 注意: 如slapd-config(5)指出的, 在 olcSyncRepl 指示中定义的 URLs 是从它们那里复制的服务器的 URLs. 这些必须准确地匹配 slapd 监听(命令行参数选项的 -h )的URLs . 否则 slapd 可能尝试从它自身复制, 而导致循环. 

### 4.MirrorMode
镜像模式配置实际上非常容易. 如果你已经配置了一个普通的 slapd syncrepl 提供者, 那么唯一的改变就是以下两个参数: 
```
       mirrormode  on
       serverID    1
```

> 注意: 你需要确保每个镜像的serverID是不同的并且把它作为一个全球配置选项. 

#### 1.Mirror Node配置
第一步是配置syncrepl提供者，就像 配置提供者slapd 一节写的那样.

注意: Delta-syncrepl还不支持镜像模式.

这里是从一个在refreshAndPersist模式下使用LDAP同步复制的例子中截取的片断:

镜像模式节点 1: 
```
       # 全球部分
       serverID    1
       # 数据库部分
 
       # syncrepl参数
       syncrepl      rid=001
                     provider=ldap://ldap-sid2.example.com
                     bindmethod=simple
                     binddn="cn=mirrormode,dc=example,dc=com"
                     credentials=mirrormode
                     searchbase="dc=example,dc=com"
                     schemachecking=on
                     type=refreshAndPersist
                     retry="60 +"
 
       mirrormode on
```
镜像模式节点 2: 
```
       # 全球部分
       serverID    2
       # 数据库部分
 
       # syncrepl参数
       syncrepl      rid=001
                     provider=ldap://ldap-sid1.example.com
                     bindmethod=simple
                     binddn="cn=mirrormode,dc=example,dc=com"
                     credentials=mirrormode
                     searchbase="dc=example,dc=com"
                     schemachecking=on
                     type=refreshAndPersist
                     retry="60 +"
 
       mirrormode on
```
它真的很简单; 每个镜像模式节点设置得完全一样, 除了 serverID 是唯一的, 并且每个消费者都被指向另一个服务器. 

##### 1.容错配置
这通常有两个选择; 1. 硬件代理/负载均衡 或 专用的代理软件, 2. 使用一个 Back-LDAP 代理作为一个 syncrepl 提供者

一个典型的企业例子可能是: 

![图 X.Y: 在一个双数据中心配置中使用镜像模式](http://www.openldap.org/doc/admin24/dual_dc.png)

图 X.Y: 在一个双数据中心配置中使用镜像模式 

##### 2.标准消费者配置
这和设置消费者slapd一节完全一样. 可以设置一个普通的复制模式, 也可以使用 delta-syncrepl 复制模式. 

#### 2.MirrorMode总结
现在你将有一个目录架构提供单主服务器复制的全部一致性保障,同时也提供多主服务器复制的高可用性. 

### 5.Syncrepl代理

![图片 X.Y: 取代slurpd](http://www.openldap.org/doc/admin24/push-based-complete.png)

图片 X.Y: 取代slurpd 

以下例子是一个自包含的推模式复制方案: 
```
        #######################################################################
        # 标准 OpenLDAP 主/提供者（服务器）
        #######################################################################
 
        include     /usr/local/etc/openldap/schema/core.schema
        include     /usr/local/etc/openldap/schema/cosine.schema
        include     /usr/local/etc/openldap/schema/nis.schema
        include     /usr/local/etc/openldap/schema/inetorgperson.schema
 
        include     /usr/local/etc/openldap/slapd.acl
 
        modulepath  /usr/local/libexec/openldap
        moduleload  back_hdb.la
        moduleload  syncprov.la
        moduleload  back_monitor.la
        moduleload  back_ldap.la
 
        pidfile     /usr/local/var/slapd.pid
        argsfile    /usr/local/var/slapd.args
 
        loglevel    sync stats
 
        database    hdb
        suffix      "dc=suretecsystems,dc=com"
        directory   /usr/local/var/openldap-data
 
        checkpoint      1024 5
        cachesize       10000
        idlcachesize    10000
 
        index       objectClass eq
        # 其它索引
        index       default     sub
 
        rootdn          "cn=admin,dc=suretecsystems,dc=com"
        rootpw          testing
 
        # syncprov特有的索引
        index entryCSN eq
        index entryUUID eq
 
        # 主数据库的syncrepl 提供者
        overlay syncprov
        syncprov-checkpoint 1000 60
 
        # 让复制DN 有无限的搜索权限
        limits dn.exact="cn=replicator,dc=suretecsystems,dc=com" time.soft=unlimited time.hard=unlimited size.soft=unlimited size.hard=unlimited
 
        database    monitor
 
        database    config
        rootpw          testing
 
        ##############################################################################
        # 消费者代理,它通过Syncrepl拉数据并且通过slapd-ldap推数据
        ##############################################################################
 
        database        ldap
        # 忽略其他数据库的冲突, 因为我们需要推同样的后缀
        hidden              on
        suffix          "dc=suretecsystems,dc=com"
        rootdn          "cn=slapd-ldap"
        uri             ldap://localhost:9012/
 
        lastmod         on
 
        # 我们不需要对这个DSA做任何操作
        restrict        all
 
        acl-bind        bindmethod=simple
                        binddn="cn=replicator,dc=suretecsystems,dc=com"
                        credentials=testing
 
        syncrepl        rid=001
                        provider=ldap://localhost:9011/
                        binddn="cn=replicator,dc=suretecsystems,dc=com"
                        bindmethod=simple
                        credentials=testing
                        searchbase="dc=suretecsystems,dc=com"
                        type=refreshAndPersist
                        retry="5 5 300 5"
 
        overlay         syncprov
```
这种类型的一个复制配置可能是这样的: 
```
        #######################################################################
        # 标准 OpenLDAP 无 Syncrepl的从服务器
        #######################################################################
 
        include     /usr/local/etc/openldap/schema/core.schema
        include     /usr/local/etc/openldap/schema/cosine.schema
        include     /usr/local/etc/openldap/schema/nis.schema
        include     /usr/local/etc/openldap/schema/inetorgperson.schema
 
        include     /usr/local/etc/openldap/slapd.acl
 
        modulepath  /usr/local/libexec/openldap
        moduleload  back_hdb.la
        moduleload  syncprov.la
        moduleload  back_monitor.la
        moduleload  back_ldap.la
 
        pidfile     /usr/local/var/slapd.pid
        argsfile    /usr/local/var/slapd.args
 
        loglevel    sync stats
 
        database    hdb
        suffix      "dc=suretecsystems,dc=com"
        directory   /usr/local/var/openldap-slave/data
 
        checkpoint      1024 5
        cachesize       10000
        idlcachesize    10000
 
        index       objectClass eq
        # 其它索引
        index       default     sub
 
        rootdn          "cn=admin,dc=suretecsystems,dc=com"
        rootpw          testing
 
        # 让复制DN拥有无限的搜索权限
        limits dn.exact="cn=replicator,dc=suretecsystems,dc=com" time.soft=unlimited time.hard=unlimited size.soft=unlimited size.hard=unlimited
 
        updatedn "cn=replicator,dc=suretecsystems,dc=com"
 
        # 提交更新到主服务器
        updateref   ldap://localhost:9011
 
        database    monitor
 
        database    config
        rootpw          testing
```
你能看到我们在这里使用了updatedn参数,而它的示范ACLs(usr/local/etc/openldap/slapd.acl) 可能如下: 
```
        # 给复制DN无限的读权限.  这个ACL可能需要和其他ACL声明配合.
        # 
 
        access to *
             by dn.base="cn=replicator,dc=suretecsystems,dc=com" write
             by * break
 
        access to dn.base=""
                by * read
 
        access to dn.base="cn=Subschema"
                by * read
 
        access to dn.subtree="cn=Monitor"
            by dn.exact="uid=admin,dc=suretecsystems,dc=com" write
            by users read
            by * none
 
        access to *
                by self write
                by * read
```
为了支持更多的复制, 只要加入更多的数据库 ldap 节并相应增加 syncrepl rid 号码.

> 注意: 你必须以相同的数据填充主目录和从目录,而不是像使用普通Syncrepl时候那样

如果你没有修改主目录配置的权限，你可以配置一个独立的ldap代理, 它看起来像这样:

![图片 X.Y: 以一个独立版本取代slurpd](http://www.openldap.org/doc/admin24/push-based-standalone.png)

图片 X.Y: 以一个独立版本取代 slurpd 

以下配置是一个独立的LDAP代理的配置示例: 
```
        include     /usr/local/etc/openldap/schema/core.schema
        include     /usr/local/etc/openldap/schema/cosine.schema
        include     /usr/local/etc/openldap/schema/nis.schema
        include     /usr/local/etc/openldap/schema/inetorgperson.schema
 
        include     /usr/local/etc/openldap/slapd.acl
 
        modulepath  /usr/local/libexec/openldap
        moduleload  syncprov.la
        moduleload  back_ldap.la
 
        ##############################################################################
        # 消费者代理,通过Syncrepl拉数据并通过slapd-ldap推数据
        ##############################################################################
 
        database        ldap
        # ignore conflicts with other databases, as we need to push out to same suffix
        hidden              on
        suffix          "dc=suretecsystems,dc=com"
        rootdn          "cn=slapd-ldap"
        uri             ldap://localhost:9012/
 
        lastmod         on
 
        # 我们不需要对这个DSA做任何操作
        restrict        all
 
        acl-bind        bindmethod=simple
                        binddn="cn=replicator,dc=suretecsystems,dc=com"
                        credentials=testing
 
        syncrepl        rid=001
                        provider=ldap://localhost:9011/
                        binddn="cn=replicator,dc=suretecsystems,dc=com"
                        bindmethod=simple
                        credentials=testing
                        searchbase="dc=suretecsystems,dc=com"
                        type=refreshAndPersist
                        retry="5 5 300 5"
 
        overlay         syncprov
```
如你所见, 使用Syncrepl和slapd-ldap(8)剪裁你的复制来满足你的特有的网络拓扑，你可以让自己的想象力变得很疯狂. 

# 19.维护
系统管理是所有关于维护，让我们讨论如何正确维护OpenLDAP的部署才是公平。

## 1.目录备份

## 2.Berkeley DB的日志

## 3.Checkpointing
MORE/TIDY

如果您在slapd.conf中把“检查点10245”（到1024KB或5分钟后检查点，例如），这不会每5分钟检查点，你可能认为。从霍华德的解释是：

“在OpenLDAP的2.1和2.2的检查点指令的作用如下： - *时，有一个写操作*，比<查看>分钟自上次检查点发生，执行检查点多。如果有超过<查看>分钟不发生，不进行任何检查站的任何其他写操作写入数据后通过，所以它可能失去最近发生的写。'“

换句话说，写操作产生的小于“检查”分钟后，直到之后的下一个写操作发生在最后的检查点不会被检查点“检查”，因为检查点分钟过去了。

这在2.3被修改为检查点确实每隔一段时间;在此期间，一个解决方法是调用从cron脚本，每隔一段时间“db_checkpoint”，说5分钟。

## 4.Migration
版本之间迁移或升级，这取决于您的部署类型所需的最简单的步骤是：

 1. 在适当的时候停止当前服务器
 2. 用slapcat导出当前数据
 3. 清除当前的数据目录（/usr/local/var/openldap-data/）配置在DB_CONFIG中
 4. 执行软件升级
 5. slapadd将导出的数据导回目录
 6. 开启服务

显然，这并不满足任何复杂的部署，像MirrorMode或n路多主，但按照上述的部分，它可以使用商业支持和社会各界的支持应该有所帮助。同时检查故障排除部分。

# 20.Monitoring
slapd（8）支持一个可选LDAP监控接口，你可以用它来获取你的slapd实例的当前状态信息。例如，该接口可以让你确定有多少客户端连接到当前服务器。监控信息通过一个专门的后端（monitor后端）提供。slapd-monitor(5)的手册页可用。

当使能监控接口，LDAP客户端的访问和其他控制的信息可被monitor后端访问。

启用后，monitor后端动态生成和返回在`cn=Monitor`子树的搜索请求响应的对象。每个对象包含关于服务器的一个特定方面的信息。该信息中包含用户应用程序和操作属性的组合。该信息可以被ldapsearch（1）访问，使用任何通用的LDAP浏览器或专门的监测工具。在访问监测信息部分提供了有关如何使用ldapsearch（1）来访问监控信息的简短教程，而监视信息部分展示了监控信息的细节和其组织。

而对于监视后端的支持，在默认情况下slapd（8）是支持的，但这种支持需要一些配置可以变得更灵活。可以使用｀cn=config｀或slapd.conf（5）都支持。前者在本章的Monitor configuration via cn=config部分讨论。后者在本章的 Monitor configuration via slapd.conf(5)部分讨论。这些章节假设monitor后端内置的slapd（例如，--enable-monitor=yes,默认值）。如果monitor后端建成为一个模块（例如，--enable-monitor=mod，该模块必须加载。模块加载在配置的slapd和slapd配置文件的章节中讨论。

# 21.Tuning

# 22.调试
