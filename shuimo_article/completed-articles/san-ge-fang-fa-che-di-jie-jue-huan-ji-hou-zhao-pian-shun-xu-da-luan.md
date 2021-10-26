想着偷偷懒，用用一键换机功能来转移手机的三千张照片吧，结果就踩坑了，照片顺序全部被打乱，然后分析了原因，搞了一晚上才解决好。总结出来3种解决办法
## 一键换机并不完美
前段时间换了新手机，听大家说MIUI自带的一键换机APP效果不错，想着两个都是小米手机，心想照片顺序的问题应该解决了吧，就用外接硬盘的方式备份到本地（备份了80多G数据，3千多张照片），再还原到新手机。

好久之后恢复完成了，照片顺序问题还是没解决，用了照片时间修复软件还是不行，相机拍得照片时间都修复对了，微信保存的网图顺序还是不对，最致命的是微信点击相册发图顺序乱糟糟，给好友发一张图得找半天。

你说这一键换机为什么不能完美的把照片同步过来呢？！

## 换种方式来转移照片
想起来Windows系统上使用剪切命令是不会更改文件创建时间的，当时没多想，就脑抽的做了个骚操作
1. 把已还原的照片文件夹全部删除
2. 把旧手机的照片文件夹（DCIM等）通过MIUI文件管理器的移动功能移动到了外置硬盘（千万别学我，我已踩坑）
3. 再把硬盘里的文件移动到新手机 

但，踩坑了，移动过去的照片顺序仍然是乱的，查看照片的修改时间，全部被重写成当天日期了。。哭。。。心里十万头马踏尘而过。。

## 分析问题产生的原因



每个文件的属性里面都有[3个时间，分别是访问时间、修改时间、创建时间](https://blog.51cto.com/meiling/2062700 "linux 下查看文件的完整时间信息及三种时间属性:")

```>>>stat filename```命令可以看到文件的这个三个时间戳
```
access time(atime) 最后访问时间
modify time(mtime) 内容修改时间
change time(ctime) 状态修改时间
```


MIUI相册根据mtime属性来排序，ctime并不会参与。

MIUI最坑的地方在于用USB设备备份还原的时候，不管选择移动功能还是复制功能都会把mtime时间修改，这就很无奈，所以手机里的照片顺序全乱套了。

既然知道了MIUI相册的排序规则，那么就可以通过技术手段来解决了。

## 三大方法修复照片时间
自己总结出来三种比较靠谱的方法，前两种适用于手机照片较少的用户，第三种方法需要有linux命令的使用经验。

- 通过时间修复软件来修复照片顺序
- 通过MIUI相册自带的时间修复功能来修复照片顺序
- 通过linux终端命令来彻底解决所有的照片顺序问题

---

**「方法一」** 
相机拍摄的照片可以通过读取exif元数据的方式来重建mtime时间戳，用快图浏览、照片时间修复之类的软件来修复照片顺序，打开软件后，先选择需要修复照片文件所在的目录，再选择时间修复功能就可以了。

但是类似于微信保存的网图等没有exif信息的图片，这种方法就不适用了，只能用linux终端命令来修改了。

---

**「方法二」** MIUI相册提供了时间修改功能，所以在照片不多的情况下可以通使用这个方法，打开相册，点开照片，查看属性，直接点击照片时间位置，弹出修改时间的界面就可以调整了。

MIUI相册没有修改时间至小时的功能，只能修改到日，还有个问题就是miui相册修改文件日期会顺带**把文件名也改变**，不仅仅是修改了时间属性，这个需要注意下

视频时间的修改方法也同样如此，使用相册的时间修改功能。

---

**「方法三」** 该方法使用终端命令来操作（前提是旧手机中没删照片），需要两台手机处于局域网中，并分别安装```Termux```软件，没有安装此软件的看最后一章有讲配置方法。

**该方法最简单**，在配置好的手机中打开Termux软件，旧手机开启ssh服务，新手机使用```scp -p -P port user@ip-address:* *```命令，复制服务器文件(夹)到本地，直接把旧手机中包含照片的文件夹复制过来，并且保留照片的修改时间和访问时间和访问权限，实乃**杀手锏！**

这是我从旧手机中拷贝微信聊天小视频的命令，因为一键换机后我发现[微信聊天记录里的小视频](/data/data/com.termux/files/home/storage/shared/Android/data/com.tencent.mm/MicroMsg/文件夹/video "微信聊天记录的小视频存放位置为")也没了，真就处处是坑！



不过拷贝照片目录也是一样的方法，把包含不同目录的命令都执行一下即可。
```
scp -p -P 8022 admin@192.168.1.103:/data/data/com.termux/files/home/storage/shared/Android/data/com.tencent.mm/MicroMsg/4f0cd58026b0b8da26a41089d764a40a/video/* .
命令执行完后，回到微信就可以正常打开小视频了，不会提示文件已清理了。
```

复制完以后，可以手动清除下相册的用户数据来解决，但MIUI很鸡贼，不让你轻易清除用户数据，那我们可以下载第三方软件管理工具清除或者去应用商店里面把相册升级后，再卸载相册更新，也能绕过限制清除MIUI相册数据，这时候再启动相册，就会发现照片自动都索引出来了。

## 手机上如何使用Termux，以及开启SSH服务的步骤

手机上需要安装[Termux](https://wiki.termux.com/wiki/Remote_Access "Termuxd的官方wiki")软件，然后通过SSH的方式拿电脑来远程操作手机提升效率，你用手机敲命令也行。。

以下步骤两台手机都需要设置：

**「安装OpenSSH」**

```
pkg upgrade

pkg install root-repo

pkg install openssh
```


**「设置SSH密码」**

- 运行```passwd```命令，输入想设置的密码，注意输入的字符是不可见的。

- 运行```ifconfig```命令，查看wlan0的ip地址并记录下来，待会SSH远程要用。

- 运行```termux-setup-storage```命令可以使得在不root的情况下获取手机内置存储的权限，手机会提示termux获取存储空间权限，给予即可。

- 再执行```sshd```命令启动SSH服务

- 然后在电脑的PowerShell或者Terminal开始ssh远程到手机```ssh <ip address> -p 8022```

**「更改Termux的shell实现更多实用功能」**

这里使用[ohmyzsh](https://ohmyz.sh/)来替代



手机需要执行这两条命令
```
pkg  install curl git zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

根据提示是否选择替换你的当前shell为zsh  输入y即可

安装好后，现在的Termux就支持更多的扩展命令和命令自动补全了。

## 总结
这次彻底搞定了之前备份照片的致命问题，在前几年尚不熟悉Linux的时候，遇到被打乱的照片只能忍痛删除，不舍得删的照片就只好的乱七八糟排序在一起，非常不舒服。

**MIUI鸡贼**也不是一天两天了，系统上加入了各种限制，也有了更多的负优化，加入了云控系统，系统升级后经常会阉割某些好用的功能，真就什么功能好用就砍什么。

这次的折腾收获也不少，最起码对于linux的三个时间属性和scp命令比较熟悉了，以及中间折腾过程中尝试的方法，虽然文中写的方法看起来很轻松简单，但也是失败了多次才摸索出来，幸好自己的数据没丢，一切都还有拯救的机会。

