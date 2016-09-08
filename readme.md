# [zmirror](https://github.com/aploium/zmirror) 一键部署脚本

使用[zmirror](https://github.com/aploium/zmirror)快速部署镜像的脚本  

## 前置需求

1. 一台墙外VPS, OpenVZ/Xen/KVM均可  
2. 操作系统:    
    * 支持的操作系统:  
        * Ubuntu 14.04/15.04(不支持HTTP2)/15.10/16.04+  
        * Debian 8 (不支持HTTP/2)  
    * 推荐的操作系统:  
        * Ubuntu 16.04 x86_64
    * 全新(刚安装完成)的操作系统. 如果系统中有其他东西, 可能会产生冲突   
    * root权限  
3. 域名
    * 每个镜像要求一个三级域名(类似于`g.zmirrordemo.com`这样的, 有三部分, 两个点)  
    * 域名已经在DNS记录中正确指向你的VPS
  
## 运行方法

* **我没有SSL证书 (如果不懂, 请使用这个)**
    ```bash
    sudo apt-get -y update && sudo apt-get -y install python3 git
    git clone https://github.com/aploium/zmirror-onekey.git --depth=1
    cd zmirror-onekey
    sudo python3 deploy.py
    ```
    然后按照脚本给予的提示继续, 过程中会自动获取SSL证书  
    如果有不懂的, 可参考下面的安装视频  
    如果遇到bug, 请[发issues](https://github.com/aploium/zmirror-onekey/issues)提出  
  

  
* **我已有SSL证书**  
    如果已有证书, 希望使用自己提供的证书, 而不是通过 let's encrypt 获取  
    请将上面代码中的第四行替换成下面的样子, 在运行期间会提示你输入证书路径的:  
    ```bash
    sudo python3 deploy.py --i-have-cert
    ```
    
    > **警告**  
    > 不支持加密的私钥, 如果私钥有密码加密, 请先解密  


## 安装过程视频
请点击下面的图片打开  
"视频"中的文字可以被选中和复制  
[![Installation demo of zmirror-onekey](https://asciinema.org/a/85170.png)](https://asciinema.org/a/85170)

## 特性

* 支持一次部署多个镜像, 支持同VPS多镜像  
* 自动安装 [let's encrypt](https://letsencrypt.org/) 并申请证书, 启用HTTPS  
* 自动添加 let's encrypt 的定期renew脚本到crontab  
* 启用[HTTP/2](https://zh.wikipedia.org/wiki/HTTP/2) ps:Debian8和Ubuntu15.04不支持HTTP/2  
* 启用[HSTS](https://zh.wikipedia.org/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8)  

## FAQ

 1. **有没有部署完成的Demo?**  

    当然有, 请戳 [zmirror-demo](https://github.com/aploium/zmirror#demo)  

 2. **我没有自己的域名**  

    有一些提供免费(也没有注册门槛)域名的注册商  
    比如最有名的 [.tk 域名](http://www.dot.tk/) 和后面出来的 [.cf 域名](http://www.dot.cf/)  
    可以快速注册免费的域名  

 4. **安装完成后各个程序的文件夹在哪?**  

    * *zmirror*  
        安装在 `/var/www/镜像名` 文件夹下  
        镜像名为每个镜像的名字, 比如YoutubePC就是 `/var/www/youtubePC`  
    * *let's encrypt*  
        本体在: `/etc/certbot/`  
        申请到的证书位置, 请看 [certbot文档-where-are-my-certificates](https://certbot.eff.org/docs/using.html#where-are-my-certificates)
    * *Apache*  
        Apache的配置文件在`/etc/apache2/`下  
        其中各个站点的配置文件在`/etc/apache2/sites-enabled/`  
    
        Apache日志文件在`/var/log/apache2/镜像名_后缀.log`  
        后缀为 _error 的日志文件中, 同时包含了stdout的输出(无论是否是错误), 对debug会有帮助  
        

 5. **为什么安装的是Apache, 而不是Nginx, 我可以选择吗?**  
    
    因为Apache的wsgi对python更友好  
    而且Nginx没有Visual Host功能  
    在性能上, 由于性能瓶颈是zmirror本身, 所以Apache和Nginx之间的性能差距可以被忽略  
    
    目前一键脚本只能安装Apache, 不支持Nginx, 也没有支持Nginx的计划, 如果需要Nginx, 请手动部署  
    手动部署可以参考 [zmirror wiki](https://github.com/aploium/zmirror/wiki)  
    当然, 如果你能写一份Nginx部署教程, 我会很感谢的~ :)  

 6. **安装的Apache版本?**
    
    在Ubuntu中, 使用的是 PPA:ondrej/apache2 理论上应该是最新版, 或者接近最新版(2.4.23+)  
    在Debian8中, 使用系统的 apt-get 安装, 版本比较旧, 所以Debian不支持HTTP/2  

 7. **Let's encrypt 证书自动更新?**

    安装脚本会自动创建定期更新证书的脚本, 脚本位置为 `/etc/cron.weekly/zmirror-letsencrypt-renew.sh`  

 8. **证书有效期为什么只有90天?**

    主要是因为Let's encrypt认为, 证书的申请和部署可以自动化时, 90天足够了.  
    具体可以看[这个官方说明](https://community.letsencrypt.org/t/pros-and-cons-of-90-day-certificate-lifetimes/4621)(可能需要自备梯子)  
    本安装脚本会在linux定时任务(crontab)中加入自动续期的脚本, 不用担心证书过期  
    即使自动续期脚本万一失效了, let's encrypt也会在快要过期时邮件通知你  

 9. **其他高级功能, 比如说CDN, 在哪?**

    这个脚本只提供最基础的部署, 高级功能需要手动配置  
    请看[config_default.py](https://github.com/aploium/zmirror/blob/master/config_default.py)和[custom_func.sample.py](https://github.com/aploium/zmirror/blob/master/custom_func.sample.py)中的说明  
    
    如果想用CDN, 可以看这个教程[使用七牛作为zmirror镜像CDN](https://github.com/aploium/zmirror/wiki/%E4%BD%BF%E7%94%A8%E4%B8%83%E7%89%9B%E4%BD%9C%E4%B8%BAzmirror%E9%95%9C%E5%83%8F%E7%9A%84CDN)  
    
    > **警告**  
    > 如果你想要修改`config_default.py`中的某项设置, 请不要直接修改  
    > 而应该将它复制到`config.py`中, 然后修改`config.py`里的设置  
    > `config.py`中的设置会覆盖掉`config_default.py`中的同名设置  
    > 除非你是开发者, 否则无论如何都不应该修改`config_default.py`  

10. **网速太慢?**

    如果你的VPS提供商允许的话, 可以试试看[net-speeder](https://github.com/snooda/net-speeder)  
    
    或者换一个网速快的VPS, Demo站使用的VPS是[Ramnode](https://clientarea.ramnode.com/aff.php?aff=2990)  
    服务器地点是LA(Los Angeles), 速度相当快.  
    ps: 如果你也想试试看的话, 请点击[这个链接](https://clientarea.ramnode.com/aff.php?aff=2990)进入, 这里有我的推广小尾巴, 你买了的话我会有一丢丢(好像是10%)分成  
    ramnode允许使用net-speeder  

