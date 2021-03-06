## NGINX学习笔记 - HTTPS详解
**目录**
  * [NGINX学习笔记 - HTTPS详解](#nginx学习笔记---https详解)
     * [保护HTTP的安全](#保护http的安全)
     * [数字加密](#数字加密)
        * [术语](#术语)
     * [对称密钥加密技术](#对称密钥加密技术)
        * [对称加密的弊端](#对称加密的弊端)
        * [秘钥长度](#秘钥长度)
     * [公开密钥加密技术（非对称加密）](#公开密钥加密技术非对称加密)
        * [RSA](#rsa)
        * [混合加密系统和会话密钥](#混合加密系统和会话密钥)
     * [数字签名](#数字签名)
        * [签名是加了密的校验和](#签名是加了密的校验和)
        * [数字签名的步骤](#数字签名的步骤)
     * [数字证书](#数字证书)
        * [证书的主要内容](#证书的主要内容)
        * [X.509 v3证书](#x509-v3证书)
        * [用证书对服务器进行认证](#用证书对服务器进行认证)
     * [HTTPS](#https)
        * [HTTPS方案](#https方案)
        * [SSL握手](#ssl握手)
        * [服务器证书](#服务器证书)
           * [站点证书的有效性](#站点证书的有效性)
        * [自签名证书](#自签名证书)
        * [自签名证书和私有CA签名的证书的区别](#自签名证书和私有ca签名的证书的区别)
        * [扩展名及各种证书格式的区别](#扩展名及各种证书格式的区别)
           * [1.证书格式](#1证书格式)
           * [2.转换方式](#2转换方式)
        * [SSL双向认证](#ssl双向认证)
        * [浏览器导入证书的方式](#浏览器导入证书的方式)
        * [常见错误](#常见错误)
        * [总结](#总结)
        * [参考网址](#参考网址)


### 保护HTTP的安全
HTTP 的安全版本要高效、 可移植且易于管理， 不但能够适应不断变化的情况而且还应
该能满足社会和政府的各项要求。 我们需要一种能够提供下列功能的 HTTP 安全技术。
- 服务器认证（客户端知道它们是在与真正的而不是伪造的服务器通话）。
- 客户端认证（服务器知道它们是在与真正的而不是伪造的客户端通话）。
- 完整性（客户端和服务器的数据不会被修改）。
- 加密（客户端和服务器的对话是私密的， 无需担心被窃听）。
- 效率（一个运行的足够快的算法， 以便低端的客户端和服务器使用）。
- 普适性（基本上所有的客户端和服务器都支持这些协议）。
- 管理的可扩展性（在任何地方的任何人都可以立即进行安全通信）。
- 适应性（能够支持当前最知名的安全方法）。
- 在社会上的可行性（满足社会的政治文化需要）。

HTTPS 是最流行的 HTTP 安全形式。 
### 数字加密
#### 术语
在这个数字加密技术的入门介绍中，我们会讨论以下内容。

- `密钥`：改变密码行为的数字化参数。
- `对称密钥加密系统`：编 / 解码使用相同密钥的算法。
- `不对称密钥加密系统`：编 / 解码使用不同密钥的算法。
- `公开密钥加密系统`：一种能够使数百万计算机便捷地发送机密报文的系统。
- `数字签名`：用来验证报文未被伪造或篡改的校验和。
- `数字证书`：由一个可信的组织验证和签发的识别信息。

### 对称密钥加密技术
在对称密钥加密技术中， 发送端和接收端使用相同的秘钥。
![](https://github.com/melon-s/learn_nginx/blob/master/images/%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E6%8A%80%E6%9C%AF.jpg)
#### 对称加密的弊端
在传统的对称密钥加密技术中， 对小型的、 不太重要的事务来说， 40 位的密钥就足
够安全了。 但现在的高速工作站就可以将其破解， 这些工作站每秒可以进行数十亿
次计算。
#### 秘钥长度
对于对称密钥加密技术， 128 位的密钥被认为是非常强大的。 实际上，长密钥对密码安全有着非常重要的影响， 美国政府甚至对使用长密钥的加密软件实施了出口控制， 以防止潜在的敌对组织创建出美国国家安全局（National SecurityAgency， NSA） 自己都无法破解的秘密代码。
![](https://github.com/melon-s/learn_nginx/blob/master/images/%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%A0%B4%E8%A7%A3%E6%97%B6%E9%97%B4.jpg)
### 公开密钥加密技术（非对称加密）
公开密钥加密技术没有为每对主机使用单独的加密 / 解密密钥， 而是使用了两个`非对称密钥`： 一个用来对主机报文编码， 另一个用来对主机报文解码。 
#### RSA
RSA算法是目前流行的公开秘钥加密算法。
#### 混合加密系统和会话密钥
公开密钥加密算法的计算可能会很慢。 实际上它混合使用了对称和非对称策略。
比如， 比较常见的做法是在两节点间通过便捷的公开密钥加密技术建立起安全通信，然后再用那条安全的通道产生并发送临时的随机对称密钥， 通过更快的对称加密技术对其余的数据进行加密。
### 数字签名
用加密系统对报文进行签名（sign）， 以说明是谁编写的报文， 同时证明报文未被篡改过。 这种技术被称为**数字签名**（digital signing）。
#### 签名是加了密的校验和
使用数字签名有以下两个好处。
- **签名可以证明是作者编写了这条报文**。 只有作者才会有最机密的私有密钥， 因此，只有作者才能计算出这些校验和。 校验和就像来自作者的个人“签名” 一样。
- **签名可以防止报文被篡改**。 如果有恶意攻击者在报文传输过程中对其进行了修改，校验和就不再匹配了。由于校验和只有作者保密的私有密钥才能产生，所以攻击者无法为篡改了的报文伪造出正确的校验码。

**私有秘钥加密的校验和，公钥解密，篡改报文后无法计算出正确的校验和，可以将作者的私有密钥当作一种“指纹” 使用。**

#### 数字签名的步骤
- 节点 A 将变长报文提取为定长的摘要。
- 节点 A 对摘要应用了一个“签名” 函数，这个函数会将用户的私有密钥作为参数。因为只有用户才知道私有密钥，所以正确的签名函数会说明签名者就是其所有者。在下图中，由于解码函数D中包含了用户的私有密钥，所以我们将其作为签名函数使用。
- 一旦计算出签名， 节点 A 就将其附加在报文的末尾， 并将报文和签名都发送给 B。
![](https://github.com/melon-s/learn_nginx/blob/master/images/%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%E7%9A%84%E6%AD%A5%E9%AA%A4.jpg)
- 在接收端， 如果节点B需要确定报文确实是节点A写的，而且没有被篡改过，节点B就可以对签名进行检查。节点B接收经私有密钥扰码的签名，并应用了使用公开密钥的反函数。如果拆包后的摘要与节点B自己的摘要版本不匹配，要么就是报文在传输过程中被篡改了，要么就是发送端没有节点A的私有密钥（也就是说它不是节点 A）。
### 数字证书
`数字证书`（通常被称作“certs”） 中包含了由某个受信任组织担保的用户或公司的相关信息。
#### 证书的主要内容
数字证书中还包含一组信息， 所有这些信息都是由一个官方的`“证书颁发机构(CA)”`以数字方式签发的。
常见的内容， 比如：
- 对象的名称（人、 服务器、 组织等） ；
- 过期时间；
- 证书发布者（由谁为证书担保） ；
- 来自证书发布者的数字签名。

任何人都可以创建一个数字证书，但并不是所有人都能够获得受人尊敬的签发权，从而为证书信息担保，并用其私有密钥签发证书。

典型的证书结构如图:
![](https://github.com/melon-s/learn_nginx/blob/master/images/%E5%85%B8%E5%9E%8B%E7%9A%84%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%E6%A0%BC%E5%BC%8F.jpg)
#### X.509 v3证书
大多数证书都以一种标准格式—`X.509 v3`， 来存储它们的信息。

X.509 证书中的字段信息:
![](https://github.com/melon-s/learn_nginx/blob/master/images/X.509%E8%AF%81%E4%B9%A6%E5%AD%97%E6%AE%B5.jpg)
#### 用证书对服务器进行认证
通过 HTTPS 建立了一个安全 Web 事务之后，现代的浏览器都会自动获取所连接服 务器的数字证书。如果服务器没有证书，安全连接就会失败。

浏览器收到证书时会对签名颁发机构进行检查。如果这个机构是个很有权威的公共签名机构，浏览器可能已经知道其公开密钥了(浏览器会预先安装很多签名颁发机构的证书)。

如果对签名颁发机构一无所知，浏览器就无法确定是否应该信任这个签名颁发机构， 它通常会向用户显示一个对话框，看看他是否相信这个签名发布者。签名发布者可 能是本地的 IT 部门或软件厂商。
![](https://github.com/melon-s/learn_nginx/blob/master/images/%E9%AA%8C%E8%AF%81%E7%AD%BE%E5%90%8D%E6%98%AF%E7%9C%9F%E7%9A%84.jpg)
### HTTPS
HTTPS 是最流行的 HTTP 安全形式。它是由网景公司首创的，所有主要的浏览器和 服务器都支持此协议。

使用 HTTPS 时，所有的 HTTP 请求和响应数据在发送到网络之前，都要进行加密。 HTTPS 在 HTTP 下面提供了一个传输级的密码安全层——可以使用 SSL，也可以使用其后继者—— 传输层安全(Transport Layer Security，TLS)。由于 SSL 和 TLS 非常类似，所以我们不太严格地用术语 SSL 来表示 SSL 和 TLS。
![](https://github.com/melon-s/learn_nginx/blob/master/images/HTTP%E4%BC%A0%E8%BE%93%E5%B1%82%E5%AE%89%E5%85%A8.jpg)
#### HTTPS方案
现在，安全 HTTP 是可选的。请求一个客户端(比如 Web 浏览器)对某 Web 资源执行某事务时，它会去检查 URL 的方案：

- 如果URL的方案为http，客户端就会打开一条到服务器端口80(默认情况下) 的连接，并向其发送老的 HTTP 命令。
- 如果URL的方案为https，客户端就会打开一条到服务器端口443(默认情况下) 的连接，然后与服务器“握手”，以二进制格式与服务器交换一些 SSL 安全参数， 附上加密的 HTTP 命令。

SSL 是个二进制协议，与 HTTP 完全不同，其流量是承载在另一个端口上的(SSL 通常是由端口 443 承载的)。如果 SSL 和 HTTP 流量都从端口 80 到达，大部分 Web 服务器会将二进制 SSL 流量理解为错误的 HTTP 并关闭连接。将安全服务进一步整 合到 HTTP 层中去就无需使用多个目的端口了，在实际中这样不会引发严重的问题。
#### SSL握手
开始加密通信之前，客户端和服务器首先必须建立连接和交换参数，这个过程叫做握手（handshake）。假定客户端叫做爱丽丝，服务器叫做鲍勃，整个握手过程可以用下图说明。
![](https://github.com/melon-s/learn_nginx/blob/master/images/SSL%E6%8F%A1%E6%89%8B%E8%BF%87%E7%A8%8B.png)

握手阶段分成五步：
- 爱丽丝给出协议版本号、一个客户端生成的 随机数（Client random），以及客户端支持的加密方法。
- 鲍勃确认双方使用的加密方法，并给出数字证书、以及一个 服务器生成的随机数（Server random）。
- 爱丽丝确认数字证书有效，然后生成一个新的 随机数（Premaster secret），并使用数字证书中的公钥，加密这个随机数，发给鲍勃。
- 鲍勃使用自己的私钥，获取爱丽丝发来的随机数（即Premaster secret）。
- 爱丽丝和鲍勃根据约定的加密方法，使用前面的三个随机数，生成 对话密钥（session key），用来加密接下来的整个对话过程。

#### 服务器证书
SSL 支持双向认证，将服务器证书承载回客户端，再将客户端的证书回送给服务器。 而现在，浏览时并不经常使用客户端证书。大部分用户甚至都没有自己的客户端证书。服务器可以要求使用客户端证书，但实际中很少出现这种情况。

##### 站点证书的有效性
SSL 自身不要求用户检查 Web 服务器证书，但大部分现代浏览器都会对证书进行简 单的完整性检查，并为用户提供进行进一步彻查的手段。网景公司提出的一种 Web 服务器证书有效性算法是大部分浏览器有效性验证技术的基础。验证步骤如下所述：

- **日期检测** 首先，浏览器检查证书的起始日期和结束日期，以确保证书仍然有效。如果证书 过期了，或者还未被激活，则证书有效性验证失败，浏览器显示一条错误信息。
- **签名颁发者可信度检测** 每个证书都是由某些 证书颁发机构(CA) 签发的，它们负责为服务器担保。证书有不同的等级，每种证书都要求不同级别的背景验证。比如，如果申请某个电 子商务服务器证书，通常需要提供一个营业的合法证明。
任何人都可以生成证书，但有些 CA 是非常著名的组织，它们通过非常清晰的流 程来验证证书申请人的身份及商业行为的合法性。因此，浏览器会附带一个签名颁发机构的受信列表。如果浏览器收到了某未知(可能是恶意的)颁发机构签发的证书，那它通常会显示一条警告信息。
- **签名检测** 一旦判定签名授权是可信的，浏览器就要对签名使用签名颁发机构的公开密钥， 并将其与校验码进行比较，以查看证书的完整性。
- **站点身份检测** 为防止服务器复制其他人的证书，或拦截其他人的流量，大部分浏览器都会试着 去验证证书中的域名与它们所对话的服务器的域名是否匹配。服务器证书中通常 都包含一个域名，但有些 CA 会为一组或一群服务器创建一些包含了服务器名称 列表或通配域名的证书。如果主机名与证书中的标识符不匹配，面向用户的客户 端要么就去通知用户，要么就以表示证书不正确的差错报文来终止连接。

#### 自签名证书
- 给nginx自签名证书

```bash
#!/bin/sh

# create self-signed server certificate:

# 1.输入服务器域名
read -p "> Enter your domain [www.example.com]: " DOMAIN

# 2.生成服务端私钥
echo "> Creating server private key (*.key) ..."
openssl genrsa -des3 -out $DOMAIN.key 2048

# 3.生成服务端证书签名请求
echo "> Creating server certificate signing request (*.csr) ..."
SUBJECT="/C=CN/ST=ZheJiang/L=HangZhou/O=Melon/OU=Melon/CN=$DOMAIN"
openssl req -new -subj $SUBJECT -key $DOMAIN.key -out $DOMAIN.csr

# 4.移除私钥中的口令，否则nginx引用此文件时需要输入密码
echo "> Removing passwd from server private key ..."
openssl rsa -in $DOMAIN.key -out $DOMAIN.key

# 5.使用私钥对签名请求进行签名
echo "> Signing certificate (*.crt) ..."
openssl x509 -req -days 3650 -in $DOMAIN.csr -signkey $DOMAIN.key -out $DOMAIN.crt

echo "TODO:"
echo "Copy $DOMAIN.crt to /usr/local/nginx/ssl/$DOMAIN.crt"
echo "Copy $DOMAIN.key to /usr/local/nginx/ssl/$DOMAIN.key"
echo "Add configuration in nginx:"
echo "server {"
echo "    ..."
echo "    listen 443 ssl;"
echo "    ssl_certificate     /usr/local/nginx/ssl/$DOMAIN.crt;"
echo "    ssl_certificate_key /usr/local/nginx/ssl/$DOMAIN.key;"
echo "}"
```

- 通过自签名CA颁发证书

```
#!/bin/bash

#创建私有CA，然后用该CA给证书签名

read -p "> Rereate CA certificate ? (y/N): " CREATE_CA_CERT
if [ "$CREATE_CA_CERT" = "Y" ] || [ "$CREATE_CA_CERT" = "y" ]; then

# 1.创建CA私钥
echo "> Creating CA private key (mCA.key) ..."
openssl genrsa -des3 -out mCA.key 2048

# 2.生成CA的自签名证书
echo "> Creating CA certificate (mCA.crt) ..."
CASUBJECT="/C=CN/ST=ZheJiang/L=HangZhou/O=GeekMelonLtd/CN=GeekMelonCA"
openssl req -new -x509 -days 3650 -subj $CASUBJECT -key mCA.key -out mCA.crt 

fi

echo "> Begin to create server certificate : "
read -p "Enter your domain [www.example.com]: " DOMAIN

# 3.生成服务端私钥
echo "> Creating server private key (*.key) ..."
openssl genrsa -des3 -out $DOMAIN.key 2048

# 4.移除私钥中的口令，否则nginx引用此文件时需要输入密码
echo "> Removing passwd from server private key ..."
openssl rsa -in $DOMAIN.key -out $DOMAIN.key

# 5.生成服务端证书签名请求
echo "> Creating server certificate signing request (*.csr) ..."
SUBJECT="/C=CN/ST=ZheJiang/L=HangZhou/O=GeekMelon/CN=$DOMAIN"
openssl req -new -subj $SUBJECT -key $DOMAIN.key -config /etc/pki/tls/openssl.cnf -extensions v3_req -out $DOMAIN.csr
# openssl req -new -key $DOMAIN.key -config /etc/pki/tls/openssl.cnf -extensions v3_req -out $DOMAIN.csr

# 6.使用CA证书给签名请求进行签名
echo "> Signing certificate with CA (*.crt) ..."
# openssl x509 -req -days 3650 -in $DOMAIN.csr -CA mCA.crt -CAkey mCA.key -CAcreateserial -out $DOMAIN.crt
openssl x509 -req -days 3650 -in $DOMAIN.csr -extfile /etc/pki/tls/v3.ext -CA mCA.crt -CAkey mCA.key -CAcreateserial -out $DOMAIN.crt

# 7.创建客户端证书
read -p "> Create client certificate ? (y/N): " CREATE_CLIENT_CERT
if [ "$CREATE_CLIENT_CERT" = "Y" ] || [ "$CREATE_CLIENT_CERT" = "y" ]; then
    echo "> Creating client private key (*.key) ..."
    openssl genrsa -des3 -out client.key 2048

    echo "> Creating client certificate signing request (*.csr) ..."
    SUBJECT="/C=CN/ST=ZheJiang/L=HangZhou/O=GeekMelon/CN=client"
    openssl req -new -subj $SUBJECT -key client.key -config /etc/pki/tls/openssl.cnf -extensions v3_req -out client.csr

    echo "> Signing certificate with CA (*.crt) ..."
    openssl x509 -req -days 3650 -in client.csr -extfile /etc/pki/tls/v3.ext -CA mCA.crt -CAkey mCA.key -CAcreateserial -out client.crt

    # 转换证书格式为window支持的格式
    echo "> Converting client cert to pfx format ..."
    openssl pkcs12 -export -inkey client.key -in client.crt -out client.pfx
fi

echo "TODO:"
echo "Copy $DOMAIN.crt to /usr/local/nginx/ssl/$DOMAIN.crt"
echo "Copy $DOMAIN.key to /usr/local/nginx/ssl/$DOMAIN.key"
echo "Add configuration in nginx:"
echo "server {"
echo "    ..."
echo "    listen 443 ssl;"
echo "    ssl_certificate     /usr/local/nginx/ssl/$DOMAIN.crt;"
echo "    ssl_certificate_key /usr/local/nginx/ssl/$DOMAIN.key;"
echo "    ..."
echo "    ssl_verify_client on;"
echo "    ssl_client_certificate /usr/local/nginx/ssl/mCA.crt;"
echo "}"
```

#### 自签名证书和私有CA签名的证书的区别
自签名的证书无法被吊销，CA签名的证书可以被吊销 能不能吊销证书的区别在于，如果你的私钥被黑客获取，如果证书不能被吊销，则黑客可以伪装成你与用户进行通信。

如果你的规划需要创建多个证书，那么使用私有CA的方法比较合适，因为只要给所有的客户端都安装了CA的证书，那么以该证书签名过的证书，客户端都是信任的，也就是安装一次就够了。如果你直接用自签名证书，你需要给所有的客户端安装该证书才会被信任，如果你需要第二个证书，则还的挨个给所有的客户端安装证书2才会被信任。

#### 扩展名及各种证书格式的区别
- .crt 证书文件 ，可以是DER（二进制）编码的，也可以是PEM（ ASCII (Base64) ）编码的 ，在类unix系统中比较常见 
- .cer 也是证书  常见于Windows系统  编码类型同样可以是DER或者PEM的，windows 下有工具可以转换crt到cer
- .csr 证书签名请求   一般是生成请求以后发送给CA，然后CA会给你签名并发回证书
- .key  一般公钥或者密钥都会用这种扩展名，可以是DER编码的或者是PEM编码的  查看DER编码的（公钥或者密钥）的文件的命令为 openssl rsa -inform DER  -noout -text -in  xxx.key  查看PEM编码的（公钥或者密钥）的文件的命令为 openssl rsa -inform PEM   -noout -text -in  xxx.key  
- .p12 证书  包含一个X509证书和一个被密码保护的私钥

**.cer/.crt是用于存放证书，它是2进制形式存放的，不含私钥。**   
**.pem跟crt/cer的区别是它以Ascii来表示。**   
**.pfx/.p12用于存放个人证书/私钥，他通常包含保护密码，2进制方式。**   

##### 1.证书格式
- **PEM 格式**    
PEM格式通常用于数字证书认证机构（Certificate Authorities，CA），扩展名为.pem, .crt, .cer, and .key。内容为Base64编码的ASCII码文件，有类似"-----BEGIN CERTIFICATE-----" 和 "-----END CERTIFICATE-----"的头尾标记。服务器认证证书，中级认证证书和私钥都可以储存为PEM格式（认证证书其实就是公钥）。Apache和类似的服务器使用PEM格式证书。  
- **DER 格式**    
DER格式与PEM不同之处在于其使用二进制而不是Base64编码的ASCII。扩展名为.der，但也经常使用.cer用作扩展名，所有类型的认证证书和私钥都可以存储为DER格式。Java使其典型使用平台。   
- **PKCS#7/P7B 格式**   
PKCS#7 或 P7B格式通常以Base64的格式存储，扩展名为.p7b 或 .p7c，有类似BEGIN PKCS7-----" 和 "-----END PKCS7-----"的头尾标记。PKCS#7 或 P7B只能存储认证证书或证书路径中的证书（就是存储认证证书链，本级，上级，到根级都存到一个文件中）。不能存储私钥，Windows和Tomcat都支持这种格式。   
- **PKCS#12/PFX 格式**    
PKCS#12 或 PFX格式是以加密的二进制形式存储服务器认证证书，中级认证证书和私钥。扩展名为.pfx 和 .p12，PXF通常用于Windows中导入导出认证证书和私钥。    

##### 2.转换方式    
- 从pfx格式的证书提取出密钥和证书
```
openssl pkcs12 -in my.pfx -nodes -out server.pem
openssl rsa -in server.pem -out server.key 
openssl x509 -in server.pem -out server.crt
```
- PEM格式的证书与DER格式的证书的转换
```
openssl x509 -in cert.pem -inform PEM -out cert.der -outform DER 
openssl x509 -in ca.cer -inform DER -out ca.pem -outform  PEM
```

#### SSL双向认证
通常情况下，只需要验证服务端证书，保证服务端的可信。但是在一些特殊的场景，比如金融行业，需要校验客户端的合法性，持有CA签署的证书的客户端才能接入服务端，此时就需要配置双向认证。   
按照CA签署服务端证书相同的方式，签署客户端证书，然后转换成window格式，导入到客户端。
```
ssl_verify_client on;
ssl_client_certificate /usr/local/openresty/nginx/ssl/mCA.crt;
```

#### 浏览器导入证书的方式
**1. 导入根证书**    
双击mCA.crt -> 点击“安装证书” -> 点击“下一步” -> 选择“将所有证书放入下列存储” -> 浏览，选择“受信任的根证书颁发机构” -> 点击“确定” -> 点击“下一步” -> 点击“完成”

**2. 导入客户端证书**    
双击client.pfx -> 下一步 -> 输入创建根证书时创建的密码 -> 选择位置 -> 完成

**3. 查看证书导入情况**   
cmd + R :  certmgr.msc

#### 常见错误
1. 自签CA签发证书，在Chrome浏览器提示“不是私密连接”
错误原因参见：[帮助](https://support.google.com/chrome/a/answer/7391219?hl=zh-Hans)  
解决方案 参见：[OpenSSL自签发配置有多域名或ip地址的证书](http://blog.csdn.net/u013066244/article/details/78725842)

#### 总结
1. 证书中包含公钥信息。
2. 私钥用于签名，相当于指纹，公钥解签。CA证书用私钥给待颁发的证书签名，客户端预安装受信根证书，可校验服务端证书不是伪造，因为可以用CA证书的公钥验证证书确实为CA签发。
3. 双向认证，服务端配置CA证书，客户端发送签发的证书到服务端，服务端可校验其合法性。这样只有CA签发的客户端才有权限访问服务端。

#### 参考网址
[1. SSL/TLS原理详解](https://segmentfault.com/a/1190000002554673)   
[2. SSL/TLS协议运行机制的概述](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)    
[3. 使用 OpenSSL 生成自签名证书](https://www.ibm.com/support/knowledgecenter/zh/SSWHYP_4.0.0/com.ibm.apimgmt.cmc.doc/task_apionprem_gernerate_self_signed_openSSL.html)  
[4. OpenSSL 与 SSL 数字证书概念贴](http://seanlook.com/2015/01/15/openssl-certificate-encryption/)  
[5. 基于OpenSSL自建CA和颁发SSL证书](http://seanlook.com/2015/01/18/openssl-self-sign-ca/)  
[6. 自签名证书和私有CA签名的证书的区别](http://blog.csdn.net/sdcxyz/article/details/47220129)   
[7. HTTPS自签发CA证书](https://yi-love.github.io/blog/%E7%BD%91%E7%BB%9C%E5%AE%89%E5%85%A8/2017/07/15/https-ca.html)   
[8. ngx_http_ssl_module](http://nginx.org/en/docs/http/ngx_http_ssl_module.html)    
[9. OpenSSL自签发配置有多域名或ip地址的证书](http://blog.csdn.net/u013066244/article/details/78725842)   
[10. OpenSSL SAN 证书](http://liaoph.com/openssl-san/)