---
title: Openssl -  微信支付证书pfx分解成pem
tags:
  - 微信
id: '152'
categories:
  - 后端
date: 2019-08-30 10:39:12
---

事件起因：做香港本地微信支付（香港公司收取香港用户钱包）申请的商户只提供了cert.pem和一个pfx的文件。程序使用pem需要cert和key两个文件，所以需要从pfx文件中提取cert和key文件。 下载OpenSSL：http://gnuwin32.sourceforge.net/packages/openssl.htm 解压Binaries.zip中openssl.exe，可单独运行 打开CMD窗口 进入OpenSSL安装目录 输入命令 ：openssl 进入openssl模式，把xxx.pfx文件放到openssl安装目录中，开始转换文件 提取cert证书：

```
pkcs12 -in xxxxx.pfx -clcerts -nokeys -out cert.pem
```

提取证书key：

```
pkcs12 -in xxxxx.pfx -nocerts -out key.pem
```

过程中需要输入证书密码 微信证书密码默认是商户号， 提取证书key的时候需要设置新密码，如果不需要设置新密码 则运行：

```
pkcs12 -in xxxxx.pfx -nocerts -nodes -out key.pem
```

在openssl安装目录里就多了两个新文件了 copy使用即可 附 openssl  pkcs12参数选项：

### 用途：

pkcs12文件工具，能生成和分析pkcs12文件。PKCS#12文件可以被用于多个项目，例如包含Netscape、 MSIE 和 MS Outlook。

### 用法：

openssl pkcs12 \[-export\] \[-chain\] \[-inkey filename\] \[-certfile filename\] \[-CApath arg\] \[-CAfile arg\]   
\[-name name\] \[-caname name\] \[-in filename\] \[-out filename\] \[-noout\] \[-nomacver\] \[-nocerts\]   
\[-clcerts\] \[-cacerts\] \[-nokeys\] \[-info\] \[-noiter\] \[-maciter\] \[-nomaciter\] \[-nomac\] \[-twopass\] \[-descert\]   
\[-certpbe alg\] \[-keypbe alg\] \[-macalg digest\] \[-keyex\] \[-keysig\] \[-password arg\] \[-passin arg\] \[-passout arg\]   
\[-rand file(s)\] \[-LMK\] \[-CSP name\]\[-engine e\] \[-des\] \[-des3\] \[-aes128\] \[-aes192\] \[-aes256\] \[-idea\]   
\[-camellia128\] \[-camellia192\] \[-camellia256\] \[-nodes\]

### 选项说明：

PKCS#12文件在创建或分析的时候要依赖有一些选项。默认的是分析一个PKCS#12文件。一个PKCS#12文件可以用B<-export>选项来创建。

### 分析选项：

\-in filename：指定私钥和证书读取的文件，默认为标准输入。必须为PEM格式。 -out filename：指定输出的pkcs12文件，默认为标准输出。 -password arg：指定导入导出口令来源。 -passin arg：输入文件保护口令来源。 -passout arg：指定所有输出私钥保护口令来源。 -noout：不打印参数编码的版本信息。 -clcerts：仅仅输出客户端证书，不输出CA证书。 -cacerts：仅仅输出CA证书，不输出客户端证书。 -nocerts：不输出任何证书。 -nokeys：不输出任何私钥信息值。 -info：输出PKCS#12文件结构的附加信息值。例如用的算法信息以及迭代次数。 -des：在输出之前用DES算法加密私钥值。 -des3：在输出之前用3DES算法加密私钥值。此项为默认。 -idea：在输出之前用IDEA算法加密私钥值。 -aes128、-aes192、-aes256：在输出之前用AES算法加密私钥值。 -camellia128、-camellia192、-camellia256：在输出之前用camellia算法加密私钥值。 -nodes：一直对私钥不加密。 -nomacver：读取文件时不验证MAC值的完整性。 -twopass：需要用户分别指定MAC口令和加密口令。 文件创建选项： -export：这个选项指定了一个PKCS#12文件将会被创建。 -in filename：指定私钥和证书读取的文件，默认为标准输入。必须为PEM格式。 -out filename：指定输出的pkcs12文件，默认为标准输出。 -inkey filename：指定私钥文件的位置。如果没有被指定，私钥必须在-in filename中指定。 -name name：指定证书以及私钥的友好名字。当用软件导入这个文件时，这个名字将被显示出来。 -certfilefilename：添加filename中所有的证书信息值。 -caname name：指定其它证书的友好名字。这个选项可以被用于多次。 -password arg：指定导入导出口令来源。 -passin arg：输入文件保护口令来源。 -passout arg：指定所有输出私钥保护口令来源。 -chain：如果这个选项被提出，则添加用户证书的证书链。标准CA中心用它来搜索。如果搜索失败，则是一个重大的错误。 -descert：用3DES对PKCS12进行加密，这样杨浦可能会导致PKCS12文件被一些“export grade”软件不能够读取。默认的是用3DES对私钥文件进行加密，用40位的RC2对证书公钥文件进行加密。 -certpbealg：该选项允许指定证书的加密算法。任何PKCS#5 v1.5或 PKCS#12 PBE类型的算法都可以被使用。默认使用的是40位的RC2。 -keypbe alg：该选项允许指定证书私钥的加密算法。任何PKCS#5 v1.5或 PKCS#12 PBE类型的算法都可以被使用。默认使用的是3DES。 -keyex：设置私钥仅仅被用于密钥交换。 -keysig：设置私钥仅仅被用于数字签名。 -macalg digest：指定MAC摘要算法。如果没有被指定，则默认使用sha1。 -nomaciter、-noiter：这个选项影响MAC值和密钥算法的迭代次数。除非你希望产生的文件能与MSIE 4.0相兼容，可以把这两个选项丢弃一边。 -maciter：加强完整性保护，多次计算MAC。 -nomac：不去规定MAC值的完整性。 -rand file(s)：指定随机数种子文件，多个文件间用分隔符分开，windows用“;”，OpenVMS用“,“，其他系统用“：”。 -CApatharg：指定CA目录。该目录必须是一个标准证书目录：每个CA文件的文件名为XXXX.0，XXXX为其持有者摘要值。 -CAfile arg：指定CA文件。  -LMK：添加本地的机械属性到私钥中。 -CSP name：微软的CSP的名字。 -engine id：指定硬件引擎。

### 注意：

有一大部分选项将不会使用。对PKCS#12文件来说，仅仅B<-in>和 B<-out>需要被使用。对文件创建来说，B<-export>和 B<-name>使用。 如果B<-clcerts>、B<-cacerts>或 B<-nocerts>没有存在，所有的证书将会被依附到PKCS#12文件将会被输出。没有保证的是提出的第一个证书不一定匹配私钥。某些软件需要一个私钥和证书，来承担在文件中的第一个证书匹配私钥：经常没有这种情况。用B<-clcerts>选项能够解决问题，因为这个选项要输出与私钥文件相匹配的证书。如果CA证书是必需的，用B<-nokeys -cacerts>选项来输出文件，仅仅输出CA证书。 B<-keypbe> 和 B<-certpbe>算法允许明确的私钥加密算法和证书的详细说明。通常的默认是好的，但是偶尔的软件不能够处理经过3DES加密了的私钥。选项B<-keypbe PBE-SHA1-RC2-40>可以被用于减少40字节的RC2私钥加密。一个完整的所有算法的描述包含在B手册中。

### 实例：

分析一个PKCS#12文件和输出到文件中： openssl pkcs12 -in file.p12 -out file.pem 仅仅输出客户端证书到文件中： openssl pkcs12 -in file.p12 -clcerts -out file.pem 不加密私钥文件： openssl pkcs12 -in file.p12 -out file.pem -nodes 打印PKCS#12格式的信息值： openssl pkcs12 -in file.p12 -info -noout 生成pkcs12文件，但不包含CA证书： openssl pkcs12 -export -inkey ocspserverkey.pem -in ocspservercert.pem  -out ocspserverpkcs12.pfx 生成pcs12文件，包含CA证书： openssl pkcs12 -export -inkey ocspserverkey.pem -in ocspservercert.pem -CAfile demoCA/cacert.pem -chain -out ocsp1.pfx 将pcks12中的信息分离出来，写入文件： openssl pkcs12 –in ocsp1.pfx -out certandkey.pem 显示pkcs12信息： openssl pkcs12 –in ocsp1.pfx -info

### BUGS：

有一些争论是PKCS#12的标准是一个大的BUG。 OpenSSL 0.9.6a以前的版本在PKCS#12的产生过程中有一个bug。在稀有的情况下用一个无效的密钥来加密并产生PKCS#12文件。这样做的结果是一些PKCS#12文件被其它的应用程序（MSIE或Netscape）会触发这个bug。因为他们不会被OpenSSL或相似的OpenSSL在产生PKCS#12文件时不能够被加密。产生的通道相当的小：在256中比1还小。 一方面这个Bug的固定影响是老的无效的加密了的PKCS#12文件不能够被固定的版本所分析。在这种情况下，B会报告’ the MAC is OK but fail with a decryption error when extracting private keys’错误。 这个问题的解决方法是用老的OpenSSL版本从PKCS#12中来提起私钥文件和证书，用新的版本、提取到的证书、私钥来重新创建PKCS#12文件。例如： old-openssl -in bad.p12 -out keycerts.pem  
openssl -in keycerts.pem -export -name "My PKCS#12 file" -out fixed.p12