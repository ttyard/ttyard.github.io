---
id: 562
title: OpenSSL 创建私有的PKI CA数字证书认证中心
date: 2016-07-04T23:04:31+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=562
permalink: '/2016/07/openssl-%e5%88%9b%e5%bb%ba%e7%a7%81%e6%9c%89%e7%9a%84pki-ca%e6%95%b0%e5%ad%97%e8%af%81%e4%b9%a6.html'
views:
  - "23"
image: /wp-content/uploads/2016/07/openssl-3-1-220x150.jpg
categories:
  - 安全
tags:
  - OpenSSL
---
## 1.简介

OpenSSL是一款开源的用于加密和认证的类库.她提供一些命令行工具,用于签发基于RSA非对称密码技术的数字证书.有时可以用她来充当简单的PKC/CA证书办法机构
  
CA即受信任的认证中心,由她维护一定范围内的信任体系.在该信任体系中的所有用户、服务器，都被发放一张数字证书来证明其身份已经被鉴定过，并为其发放一张数字证书，每次在进行数据通信的时候，通过互相检查对方的数字证书即可判别是否是本信任域中的可信体。
  
很多互联网站数据证书来加密客户端与服务器端之间的数据通信,保障数据在网络中的传输安全.这就需要拥有全球访问受信任的CA机构来为这些实体签发证书,来表明其身份.(如:VeriSign/DigiCert).
  
有些时候,我们需要构建一套用于公司内部应用程序(Apache/Nginx/OpenVPN&#8230;)使用的数字证书,用于保证内部服务器或与外网服务器之间的数据传输安全.这时我们就需要构建一套自己的PKI/CA数字证书信任体系.
  
<img src="http://images.wanglijie.cn/public/img/posts/2016/07/openssl-3.jpg" alt="openssl-3" width="300" height="115" class="aligncenter size-full wp-image-586" />

## 2.创建CA根证书

CA根证书是PKI体系中重要的组成部分,要建立树状的受信任体系,首先CA自身需要表明身份,并建立树根,其他的中级证书都依赖该根来签发证书,并通过根证书建立彼此的信任关系.CA证书由私钥Root Key和公钥Root Cert Pem组成。
  
CA根证书并不直接颁发服务器证书和客户端证书，仅用来颁发一个或多个中级证书，中级证书代理Root CA向用户和服务器颁发证书。同时，为了保证Root CA的安全，应保证Root Key处于离线并安全保存。
  
创建Root CA目录
  
选择一个目录用于存储所有的私钥和证书公钥。（/root/ca）

<pre class="prettyprint linenums">$ mkdir /root/ca
</pre>

创建目录结构，生成index.txt和serial文件，用于作为文件数据库追踪以前发的证书。

<pre class="prettyprint linenums">$ cd /root/ca
$ mkdir certs crl newcerts private
$ chmod 700 private
$ touch index.txt
$ echo 1000 &gt; serial
</pre>

**备注：**
  
certs：存放已签发的证书，newcerts：存放CA生成的新证书，private：存放私钥，crl：存放以吊销的证书，index.txt：OpenSSL定义的已签发证书的文本数据库文件，通常该文件在初始化时为空。
  
serial：签发证书时使用的序列号参考文件，该文件中的序列号为16进制数，初始化时必须提供并包含一个有效的序列号。

### 创建OpenSSL配置文件 Root CA

为openssl 创建Root CA配置文件，保存在/root/ca/openssl.cnf
  
文件中[ ca ]的部分是必须存在的，用于告诉OpenSSL CA使用的配置信息.

<pre class="prettyprint linenums">[ ca ]# `man ca`default_ca= CA_default
</pre>

在[ CA_default ]中CA的基础配置信息，请确认在此处声明的目录已经存在。

<pre class="prettyprint linenums">[ CA_default ]
# Directory and file locations.
dir               = /root/ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/ca.key.pem
certificate       = $dir/certs/ca.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_strict
</pre>

### 创建Root CA证书签名策略，作为Root CA仅用来签发中级CA证书。

<pre class="prettyprint linenums">[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
</pre>

为中级CA证书创建相对宽松的证书签名策略。中级CA证书可以为第三方终端用户和服务器签发证书。

<pre class="prettyprint linenums">[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
</pre>

在[ req ]部分，主要定义了创建证书和证书证书请求时默认选项。

<pre class="prettyprint linenums">[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca
</pre>

在[ req\_distinguished\_name ]中，定义了生成证书请求时常用的信息说明和默认值。

<pre class="prettyprint linenums">[ req_distinguished_name ]
# See &lt;https://en.wikipedia.org/wiki/Certificate_signing_request&gt;.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = GB
stateOrProvinceName_default     = England
localityName_default            =
0.organizationName_default      = Alice Ltd
#organizationalUnitName_default =
#emailAddress_default           =
</pre>

在OpenSSL中，出来基本的必要信息外，还有一些可扩展的选项可用于证书签发。

<pre class="prettyprint linenums">[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
</pre>

定义中级CA证书签发的扩展选项，其中pathlen:0是为了确保中级CA证书不能再签发新的中级CA证书。

<pre class="prettyprint linenums">[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
</pre>

在[ user_cert]中约束了签发客户端证书策略。该证书一般用于用户身份认证。

<pre class="prettyprint linenums">[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection
</pre>

在 [ server_cert ]中，定义了签发服务器证书时的证书签发策略。

<pre class="prettyprint linenums">[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
</pre>

在 [ crt_ext ]用于定义创建证书吊销列表时的策略。

<pre class="prettyprint linenums">[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always
</pre>

定义在线证书使用策略ocsp

<pre class="prettyprint linenums">[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
</pre>

### 创建Root CA Key

生成的root key必须保证其绝对的安全，并使用aes-256加密和高强度密码保护。

<pre class="prettyprint linenums">$ cd /root/ca
$ openssl genrsa  -aes256 -out private/ca.key.pem 4096
Enter pass phrase for ca.key.pem: secretpassword 
Verifying - Enter pass phrase for ca.key.pem: secretpassword

$ chmod 400 private/ca.key.pem
</pre>

备注：
  
genrsa：使用RSA算法产生私钥；-aes256：使用256位密钥的AES算法对私钥进行加密；-out：输出文件目录；4096：指定密钥的长度。

### 创建Root CA证书

使用Root Key（ca.key.pem）创建Root CA证书（ca.cert.pem）.需要为Root CA指定较长的有效期，如20年。一旦Root CA证书过期，其所有签发的证书将都不可用。
  
在使用req命令时，必须使用-config 指定配置文件，否则程序默认使用/etc/pki/tls/openssl.cnf。

<pre class="prettyprint linenums">$ cd /root/ca
openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem
Enter pass phrase for ca.key.pem: secretpassword
You are about to be asked to enter information that will be incorporated
into your certificate request.
-----
Country Name (2 letter code) [XX]:GB
State or Province Name []:England
Locality Name []:
Organization Name []:Alice Ltd
Organizational Unit Name []:Alice Ltd Certificate Authority
Common Name []:Alice Ltd Root CA
Email Address []:

$ chmod 444 certs/ca.cert.pem
</pre>

&nbsp;

### 验证签发的Root CA证书

<pre class="prettyprint linenums">$ openssl x509 -noout -text -in certs/ca.cert.pem
</pre>

确认Signature Algorithm使用的签名算法
  
确认Validity中的证书有效期
  
确认Public-Key长度
  
确认Issuer，签发者信息
  
确认Subject，Root CA证书为自签名证书，与Issuer信息相同
  
我们启用x509 v3扩展信息，在输出的文件中，还将包括一些扩展的信息，如：X509v3 Subject Key Identifier/X509v3 Authority Key Identifier/X509v3 Basic Constraints等。

### 创建中级CA证书密钥对

中级CA证书用于代替Root CA签发客户端证书和服务器证书，根证书负责签发中级CA证书并维护受信任的证书链.
  
使用中级证书主要是处于安全考虑,因为Root CA Key始终处理物理隔离并能够保障私钥的绝对安全,在中级CA证书的私钥发生泄漏后,可以通过Root CA吊销该证书,并重新签发新的中级证书密钥对.

**创建中级CA证书目录结构**
  
与Root CA类是,我们需要为中级CA证书创建用于存储证书的目录.我们将中级证书所有内容保存在/root/ca/intermediate目录下面,并创建类是Root CA的目录结构.

<pre class="prettyprint linenums">$ mkdir /root/ca/intermediate
$ cd /root/ca/intermediate
$ mkdir certs crl newcerts private
$ chmod 700 private
$ touch index.txt
$ echo 1000 &gt; serial
</pre>

在中级CA证书目录下,新建名为crlnumber的文本文件,用于保存证书吊销列表.

<pre class="prettyprint linenums">$ echo 1000 &gt; /root/ca/intermediate/crlnumber
</pre>

创建中级CA证书OpenSSL配置文件
  
将中级CA证书的配置文件保存在/root/ca/intermediate/openssl.cnf,相比Root CA配置文件修改了证书的保存目录.

<pre class="prettyprint linenums">[ CA_default ]
dir             = /root/ca/intermediate
private_key     = $dir/private/intermediate.key.pem
certificate     = $dir/certs/intermediate.cert.pem
crl             = $dir/crl/intermediate.crl.pem
policy          = policy_loose
</pre>

完整的配置文件内容如下:

<pre class="prettyprint linenums"># OpenSSL intermediate CA configuration file.
# Copy to `/root/ca/intermediate/openssl.cnf`.

[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /root/ca/intermediate
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/intermediate.key.pem
certificate       = $dir/certs/intermediate.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_loose

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See &lt;https://en.wikipedia.org/wiki/Certificate_signing_request&gt;.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = GB
stateOrProvinceName_default     = England
localityName_default            =
0.organizationName_default      = Alice Ltd
organizationalUnitName_default  =
emailAddress_default            =

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
</pre>

### 生成中级CA证书私钥

创建中级证书私钥,使用AES-256加密.

<pre class="prettyprint linenums">$ cd /root/ca
$ openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 4096
$ chmod 400 intermediate/private/intermediate.key.pem
</pre>

### 生成中级CA证书请求

使用intermediate.key.pem为生成中级CA证书,同时使用Root CA对证书进行签名.Common Name不能重复.
  
如果使用中级CA签发服务器证书和客户端证书,需要指定配置文件intermediate/openssl.cnf

<pre class="prettyprint linenums">$ cd /root/ca
$ openssl req -config intermediate/openssl.cnf -new sha256 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem
</pre>

### 生成中级CA证书

使用Root CA签发中级CA证书,并为其指定v3\_intermediate\_ca扩展属性签发证书.中级证书的有效期要短于根证书.
  
确认使用Root CA的openssl.cnf签发中级证书

<pre class="prettyprint linenums">$ cd /root/ca
$ openssl ca -config openssl.cnf -extenstions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem
$ chmod 444 intermediate/certs/intermediate.cert.pem
</pre>

注意:index.txt是openssl的文件数据库,不要直接修改或删除该文件.

### 验证中级CA证书内容

<pre class="prettyprint linenums">$ openssl x509 -noout -text -in intermediate/certs/intermediate.cert.pem
</pre>

使用Root CA根证书验证中级证书的证书链信任关系,OK表示证书链受信任.

<pre class="prettyprint linenums">$ openssl verify -CAfile certs/ca.cert.pem  intermediate/certs/intermediate.cert.pem
intermediate.cert.pem: OK
</pre>

### 创建证书链文件

当应用程序(Web浏览器)尝试验证签发的证书有效性时,会尝试验证中级CA证书和上级根证书.
  
在证书链文件中,必须同时包含根证书和中级证书.因为客户端不知知道签发的这张私有的CA根证书,你可以在所有需要访问的客户端将Root CA导入到受信任的根证书办法机构,这样该CA签发的证书就完全受信任并且证书链文件中则只用指定中级证书即可.

<pre class="prettyprint linenums">$ cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem &gt; intermediate/certs/ca-chain.cert.pem
$ chmod 444 intermediate/certs/ca-chain.cert.pem
</pre>

## 签发服务器证书和客户端证书

接下来我们将使用中级CA证书为服务器签发证书,用来加密服务器与客户端之间的数据通信.如果您在服务器使用OpenSSL已经创建过了csr证书请求,则可跳过生成Server Key的过程.

### 创建服务器证书私钥

我们在创建Root CA和intermediate CA使用了4096bit密码加密,服务器和客户端证书一般有效期为几年,可以使用2048bit加密.
  
虽然4096bit密码要更安全,但在服务器进行TLS握手和数字签名验证时会降低访问速度,因此一般Web服务器还是建议使用2048Bit密码加密.
  
如果您在创建私钥时设置了密码,则每次服务重启时都需要提供这一密码,因此不建议为服务器证书设置密码保护,可以使用aes256密码算法进行加密.

<pre class="prettyprint linenums">$ cd /root/ca
$ openssl genrsa -aes256 -out intermediate/private/r.wanglijie.cn.key.pem 2048
$ chmod 400 intermediate/private/r.wanglijie.cn.key.pem
</pre>

### 创建服务器证书请求csr

使用创建的服务器私钥生成签发证书请求csr文件.csr文件无需指定中级CA证书机构.但是必须指定Common Name,一般为域名或IP地址.

<pre class="prettyprint linenums">$ cd /root/ca
$ openssl req -config intermediate/openssl.cnf -new -sha256 -days 365 -key intermediate/private/r.wanglijie.cn.key.pem -out intermediate/csr/r.wanglijie.cn.csr.pem
</pre>

### 从证书请求csr创建服务器证书

<pre class="prettyprint linenums">$ cd /root/ca
$ openssl ca -config intermediate/openssl.cnf \
      -extensions server_cert -days 375 -notext -md sha256 \
      -in intermediate/csr/r.wanglijie.cn.cert.pem \
      -out intermediate/certs/r.wanglijie.cn.cert.pem 
$ chmod 444 intermediate/certs/r.wanglijie.cn.cert.pem
</pre>

注意：
  
在签发证书时，-extensions 选项用于指定要签发的证书类型。服务器证书使用“server\_cert ”，客户端证书使用“usr\_cert”
  
证书签发完毕后，将会在intermediate/index.txt文件中新增一条记录。

### 查看签发的证书

<pre class="prettyprint linenums">$ openssl x509 -noout -text  -in intermediate/certs/r.wanglijie.cn.cert.pem
</pre>

使用CA证书链（ca-chain.cert.pem）验证签发证书的信任关系

<pre class="prettyprint linenums">$ openssl verify -CAfile intermediate/certs/ca-chain.cert.pem intermediate/certs/r.wanglijie.cn.cert.pem
</pre>

### 部署证书

在证书签发完毕后，可以将证书部署在Nginx,Apahce或其他的应用服务器上。如果您需要将证书部署在IIS或Tomcat中，请查看下文openssl证书格式转换。

[常用数字证书格式相互转换-OpenSSL/Keytool/Jks2pfx](http://www.wanglijie.cn/?p=584)

[访问英文原文](https://jamielinux.com/docs/openssl-certificate-authority)

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [OpenSSL 创建私有的PKI CA数字证书认证中心](http://www.wanglijie.cn/2016/07/openssl-%e5%88%9b%e5%bb%ba%e7%a7%81%e6%9c%89%e7%9a%84pki-ca%e6%95%b0%e5%ad%97%e8%af%81%e4%b9%a6.html)