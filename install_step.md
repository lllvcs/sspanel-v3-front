ok，接下来就是转载之路，并在其中写一些小白遇到的坑，并如何解决的。
演示环境：CentOS 6 X64

1.首先通过 SSH 连接到远程服务器，安装 lnmp 一键包。
或者直接用宝塔面板装


wget http://soft.vpser.net/lnmp/lnmp1.2-full.tar.gz
tar xvzf lnmp1.2-full.tar.gz
cd lnmp1.2-full
./install.sh
2.请根据你的需求选择好安装组件，推荐如下。
记得自己设定的 Mysql 数据库 root 密码哦。

3.喝杯茶（我这边安装时间还是比较长的，可能是网速不好，所以稍安勿躁），等安装好。

4.然后可以看到，安装好了。


5.添加一个虚拟主机，同时创建数据库。
lnmp vhost add
进行接下来的设置,其中domain代码的是域名地址，可以去申请一个免费的域名(https://my.freenom.com/clientarea.php),记得进行dns解析

这儿输入的password和databasename和data name需要记住。
6.然后，来对 nginx 进行一些细节化配置。（我当时进行到这儿的时候是，不知道怎么修改ngix了，怎么办呢，通过ctrl+alt+f打开ftp），然后找到这个这个目录打开 /usr/local/nginx/conf/vhost/ss.panel.conf并修改。还记的第一篇说的需要安装notepad++,就用那个打开
然后添加下面这一段到 server

location / {
                        try_files $uri $uri/ /index.php$is_args$args;
                }

同时，root那一行改为

root /home/wwwroot/ss.panel/public;
ss.panel不用修改，主要是在原基础上面加一个/public

然后回到xshell就是下载程序代码。（本文所有 /home/wwwroot/ss.panel都需要你改为你自己设置的domain目录 /home/wwwroot/你的domain目录，一般默认用的域名地址作为的 ）

cd /home/wwwroot/ss.panel

yum install git -y

git clone https://github.com/glzjin/ss-panel-v3-mod.git tmp -b new_master && mv tmp/.git . && rm -rf tmp && git reset --hard

chown -R root:root *

chmod -R 777 *

chown -R www:www storage

修改完了之后，到网站目录下进行一些修改。（需要记住#后面的才是你需要复制的）

cd /home/wwwroot/ss.panel/

chattr -i .user.ini

mv .user.ini public

cd public

然后就是重新添加回权限。

chattr +i .user.ini

OK，重启一下 nginx 

service nginx restart

好，这个暂时到这里，我们待会再回来进行配置。

9.如果使用vpn才需要这一步，如果你要配置vpn，请查看原文文档。我安装的时候没有配置

10.然后我们打开 phpmyadmin ,来配置数据库了。地址为http://ip/phpmyadmin/

在ss_panel（或者你创建的数据库名字）那一栏下导入 sql 目录下的 glzjin_all.sql（https://github.com/esdeathlove/ss-panel-v3-mod 在这个github的sql里面可以找到这个文件）

导入完成，数据库这里就差不多了。

11.如果使用vpn才需要这一步，是配置vpn相关的radius。

12.然后让我们回到 ss-panel 的配置上来，
cd /home/wwwroot/ss.panel


php composer.phar install

cp config/.config.php.example config/.config.php

nano config/.config.php

ok，坑来了，nano这个其实是个编辑的命令，跟vi查不到，但是我用vi又不习惯，所以就直接通过ftp找到这个文件直接进行配置就是。
这个文件在这个目录下/home/wwwroot/你的domain目录/config，

如果你不需要注册的时候进行邮箱验证，那么请你在这配置文件里面把邮箱验证设置为false。这里面很多配置都有中文注释，你可以安装其去修改。不确定建议不改。

13.配置完了，就来创建管理员。不过这个管理的话，不会自动同步到 radius ，需要在网站上注册的才可以。
php -n xcat createAdmin
输入一个邮箱和密码，然后输入y确定就ok。如果出现添加成功，那么恭喜你搭建成功了
14.然后就是来同步一下用户。
php xcat syncusers
15.然后 crontab -e ，添加以下六段。(不需要执行这步也行)
30 22 * * * php /home/wwwroot/ss.panel/xcat sendDiaryMail
*/1 * * * * php /home/wwwroot/ss.panel/xcat synclogin
*/1 * * * * php /home/wwwroot/ss.panel/xcat syncvpn
0 0 * * * php -n /home/wwwroot/ss.panel/xcat dailyjob
*/1 * * * * php /home/wwwroot/ss.panel/xcat checkjob    
*/1 * * * * php -n /home/wwwroot/ss.panel/xcat syncnas
打开你的网站进行节点配置 ，登录后--->管理面板-->节点列表--》右下角有个+号


在使用中，有一些小注意，慢慢补充。
1、添加节点时，请注意用 " - "来分割。

前面为节点名，后面为方式。

比如 “香港 1 - Shadowsocks”


ok，到这里就结束了。
