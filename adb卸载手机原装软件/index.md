# Adb卸载手机原装软件


# Adb卸载手机原装软件

我的手机系统是miui12.5，其他手机也类似

## 准备

首先[下载google adb工具](https://developer.android.google.cn/studio/releases/platform-tools?hl=zh-cn#downloads)并解压

1. 进入开发者模式。miui中进入**设置**=》**我的****设备**=》**全部参数**。连续点击5下**miui版本**进入开发者模式。
2. 数据线链接电脑。
3. 回到**设置**中搜索框输入开发者选项，点进去找到USB调试，接受风险打开。
4. 进入adb目录打开cmd或者powershell

输入.\adb.exe shell 显示下面这样就成功链接了

![img](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/1617974989237-229f2029-ae71-441c-a3c2-bcd0d822d28b.png)

## 卸载垃圾应用

命令 ==pm uninstall –user 0 com.miui.systemAdSolution== 最后跟的是应用的包名，包名可以在应用管理中查看，想卸载哪个应用就点开那个应用，把最后的包名替换就行了。success就表示成功了。

![img](https://cdn.jsdelivr.net/gh/songzhiv/image//blog/1617976291146-d7571877-37ac-452a-aba7-6dcb137c521b.png)

<img src="https://cdn.jsdelivr.net/gh/songzhiv/image//blog/1617975324410-5e1febfd-3204-403e-9fce-fab8f8930cab.png" alt="img" style="zoom:50%;" />        

<img src="https://cdn.jsdelivr.net/gh/songzhiv/image//blog/1617975365177-b6785a7d-cc51-4eec-a4d9-93edf9275bfc.png" alt="img" style="zoom: 50%;" />

下面是我卸载的应用

```bash
pm uninstall --user 0 com.miui.systemAdSolution（小米系统广告解决方案）
pm uninstall --user 0 com.miui.analytics（小米广告分析）
pm uninstall --user 0 com.miui.personalassistant（智能助理：既不智能，也不助理，除了卡，没别哒）
pm uninstall --user 0 com.miui.bugreport  （bug 反馈：反馈了也不修，反馈个啥）
pm uninstall --user 0 com.xiaomi.gamecenter.sdk.service  （小米游戏中心服务）
pm uninstall --user 0 com.xiaomi.gamecenter  （小米游戏中心）
pm uninstall --user 0 com.miui.player  （小米音乐：垃圾）
pm uninstall --user 0 com.miui.video  （小米视：垃圾）
pm uninstall --user 0 com.android.email  （邮件：垃圾）
pm uninstall --user 0 com.miui.hybrid  （混合器）
pm uninstall --user 0 com.android.browser  （浏览器：垃圾）
pm uninstall --user 0 com.miui.yellowpage  （黄页）
pm uninstall --user 0 com.xiaomi.midrop  （小米快传）
pm uninstall --user 0 com.miui.virtualsim  （小米虚拟器）
pm uninstall --user 0 com.xiaomi.payment  （小米支付：垃圾）
pm uninstall --user 0 com.mipay.wallet  （小米钱包：垃圾）
pm uninstall --user 0 com.android.soundrecorder  （录音机）
pm uninstall --user 0 com.android.wallpaper  （壁纸：卸载完背景就成纯黑了，大家别学我）
pm uninstall --user 0 com.miui.voiceassist  （语音助手）
pm uninstall --user 0 com.miui.touchassistant  （悬浮球）
pm uninstall --user 0 com.android.cellbroadcastreceiver  （小米广播）
pm uninstall --user 0 com.xiaomi.mitunes  （小米助手）
pm uninstall --user 0 com.xiaomi.pass  （小米卡包）
pm uninstall --user 0 com.android.thememanager  （个性主题管理）
pm uninstall --user 0 com.android.wallpaper  （动态壁纸）
pm uninstall --user 0 com.android.wallpaper.livepicker  （动态壁纸获取）
pm uninstall --user 0 com.miui.klo.bugreport  （KLO bug 反馈）
```

网上说下面这四个卸载完就开不了机了，我没敢试。有兴趣的同学不妨试一下

com.miui.cloudservice（小米云服务）

com.xiaomi.account（小米账户）

com.miui.cloudbackup（云备份）

com.xiaomi.market（应用市场）

