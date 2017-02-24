---
layout: post
title:  "阿里云应用部署手记"
date:   2017-02-24
categories: 技术栈
---

softmind阿里云部署备忘

#### 阿里云安装Mogodb Ubuntu16.04

删除应用安装配置文件     [_apt代理问题会导致403_](https://yq.aliyun.com/ask/47027/?order=vote_num&p=1)
{% highlight shell %}
rm /etc/apt/apt.conf
{% endhighlight %}

添加key     [_mongodb官方教程_](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)
{% highlight shell %}
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
{% endhighlight %}

指定下载源
{% highlight shell %}
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
{% endhighlight %}

更新下载源
{% highlight shell %}
sudo apt-get update
{% endhighlight %}

安装mongodb
{% highlight shell %}
sudo apt-get install -y mongodb-org
{% endhighlight %}

在指定位置新建mongodb数据目录
{% highlight shell %}
mkdir /data
mkdir /data/db
{% endhighlight %}

启动mongod
{% highlight shell %}
sudo mongod
{% endhighlight %}

启动mongo客户端
{% highlight shell %}
mongo
{% endhighlight %}

* * *

#### 阿里云安装mysql Ubuntu16.04

更新安装源
{% highlight shell %}
sudo apt-get update
{% endhighlight %}

安装mysql（中途设置用户）
{% highlight shell %}
sudo apt install mysql-server
{% endhighlight %}

登录mysql
{% highlight shell %}
mysql -u root -p
{% endhighlight %}

* * *

#### 阿里云部署 Flask 项目

新建代码共享服务器（远端）
{% highlight shell %}
# 项目存放在~目录下
cd ~
mkdir [app name].git
cd [app name].git
# 这是一个裸仓库，专门用于存放代码
git init --bare
{% endhighlight %}

配置 git hooks 并为对应脚本文件添加执行权限 [_参考1_](http://stackoverflow.com/questions/279169/deploy-a-project-using-git-push/3387030#3387030) [_参考2_](http://www.tuicool.com/articles/3QRB7jU)
{% highlight shell %}
cd hooks
# 编辑代码更新后需要自动执行的脚本
vim post-update
# 赋予执行权限
chmod +x .git/hooks/post-update
{% endhighlight %}

初始化本地 Git 代码库 [_Flask部署_](http://blog.csdn.net/zhyh1435589631/article/details/51946439#211-配置nginx-服务器脚本) [_Git_](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000) [_Ubuntu安装最新版nodejs_](http://askubuntu.com/questions/786272/why-does-installing-node-6-x-on-ubuntu-16-04-actually-install-node-4-2-6) [_vim_](http://www.cnblogs.com/jiayongji/p/5771444.html) [_shell_](http://man.linuxde.net/cp)
{% highlight shell %}
cd [to application root]
git init
# 配置 remote 源，设置为远端的 [app name].git，并命名为 production
vim .git/config
{% endhighlight %}

配置本地库的 .gitignore [_Not working_](http://stackoverflow.com/questions/11451535/gitignore-is-not-working) [_Show ignored_](http://stackoverflow.com/questions/466764/git-command-to-show-which-specific-files-are-ignored-by-gitignore)
{% highlight shell %}
vim .gitignore
# 以下3行命令保证.gitignore生效
git rm -r --cached .
git add .
git commit -m "fixed untracked files"
# 查看指定路径下被忽略的文件或目录
git check-ignore -v *
# 添加所有文件到缓冲区
git add *
# 提交缓冲区所有文件到仓库
git commit -m 'first commit'
# 把最新的代码push到远端仓库 
git push production
{% endhighlight %}

项目部署（远端）(未包括服务器以及反向代理设置)
{% highlight shell %}
cd [to application root]
git init
# 将remote设置为softmind.git, 并命名为 code
vim .git/config
vim .gitignore
git rm -r --cached .
git add .
git commit -m "fixed untracked files"
git check-ignore -v *
git add *
git commit -m 'first commit'
# 把最新的代码抓取到本仓库
git pull code
# 新建python虚拟环境
virtualenv venv
# 安装python依赖的库
# 会遇到M2Crypto在venv环境下无法安装的问题，可在全局环境下安装，然后将二进制文件拷贝到venv对应目录下（http://stackoverflow.com/questions/10547332/install-m2crypto-on-a-virtualenv-without-system-packages）
pip install -r requirements.txt
# 进入静态文件目录
cd project/static
# 初始化npm (cnpm是淘宝npm镜像)
cnpm install
# 生产环境代码编译
cnpm run build
cd [to application root]
# 开启服务
python run.py
{% endhighlight %}

* * *




