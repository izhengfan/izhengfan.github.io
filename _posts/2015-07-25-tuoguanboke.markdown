---
layout: cnpost
title: "自建博客同时托管至 Github 与 Gitcafe"
date: 2015-07-25 19:46:00
categories: cn
tags: web Jekyll
---

__目录__

* content
{:toc}


### 博客架构

自建博客， 常见方案有 Wordpress, Octopress, Jekyll, Hexo 等。Wordpress 最是大名鼎鼎，但常被批臃肿。Octopress 不了解。Hexo 小试了一下，易用性强，不少人的评价优于 Jekyll；但我一开始使用时没法解决它对子域名的解析问题，遂弃用。最终决定用 Jekyll。Jekyll 口碑佳，且被 Github 等主流 Git 服务托管网站支持，唯一常被提及的缺点是更新时需要比 Hexo 多敲几行命令……这也不算大问题啦。用 markdown 写文章比直接写 html 不知爽快多少倍。

Jekyll 的使用，官网已够详细：[jekyllrb.com](http://jekyllrb.com/)

### 托管空间

仅个人主页和博客之用，专门租个服务器似乎不划算。明确目标：我们只需要一块能放静态网页文件的空间，让众人访问而已。动态网页啦，数据库啦，PHP 啦什么的，通通不要。

所幸，Github 现在提供 Pages 服务！而且官方支持 Jelyll 耶。注册 Github 账号，建立 `username.github.io` 项目文件夹（`username` 代表个人帐户名，下同），把网页文件往里扔，就可以通过 `username.github.io` 这个链接访问你的个人网站，So easy！

问题来了，我们不仅是地球人，还是中国人，这意味着我们还需要多考虑一堵墙的问题。目前 Github 也被丧心病狂地墙掉了，作为一个常用中文的博客，不能被墙内访问，多么寂寞。嗯，所以，能不能找个墙内备胎？还真有！Github 墙内山寨版 Gitcafe。Gitcafe 也提供 Pages 服务，也官方支持 Jekyll，实在是一个尽职的备胎。注册 Gitcafe，建立 `username` 项目文件夹，放进网页文件，一个 `username.gitcafe.io` 主页也搭好了。

Github 和 Gitcafe 网页服务的不同之处：1）前者的项目文件夹必须名为 `username.github.io`，后者则名为 `username`；2）前者使用 `master` branch，后者使用 `gitcafe-pages` branch。第二点差异会给维护添点小麻烦，后面讲。

PS：Github 除了支持 `username.github.io` 作为用户主页，也允许另外建立 `projectname` 的网页项目，可通过用户主域名的子域名访问：`username.github.io/projectname`。注意，与用户主域名项目使用 `master` branch 不同，子域名项目必须把网页文件放在 `gh-pages` branch 下。类似的， Gitcafe 也支持 `username.gitcafe.io/projectname`，一律使用 `gitcafe-pages` branch。

### 博客维护

如何同时将个人博客同步更新到 Github 和 Gitcafe 呢？ 只要同时添加两个远程代码仓库地址即可。以 Gitcafe 为例（参考[官方帮助](https://help.gitcafe.com/manuals/help/ssh-key)）。

1.假定你的电脑已安装好 Git。先为 Gitcafe 创建 SSH 密钥：

```bash
mkdir ~/.ssh #创建密钥目录
cd  ~/.ssh #进入目录
ssh-keygen -t rsa -C "YOUR_EMAIL@YOUREMAIL.COM" #创建密钥
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): #直接回车
Enter same passphrase again: #直接回车
Your identification has been saved in ~/.ssh/id_rsa.
Your public key has been saved in ~/.ssh/id_rsa.pub.
The key fingerprint is:
15:81:d2:7a:c6:6c:0f:ec:b0:b6:d4:18:b8:d1:41:48 YOUR_EMAIL@YOUREMAIL.COM
cat id_rsa.pub
```
	
最后一步会打印出你创建的公钥，复制，进入 Gitcafe 的账户设置项，粘贴为新的 SSH 公钥。

然后测试下是否可以链接到 Gitcafe 了：

```bash
ssh -T git@gitcafe.com
```

第一次连接可能出现警告，无所谓，按提示输入 yes 就好。最后你会收到成功提示：

    Hi USERNAME! You've successfully authenticated, but GitCafe does not provide shell access.

2.接下来建立项目。先到 Gitcafe 网站上建立和帐户名同名的项目。随后在本地，新建一个文件夹，并进入，执行以下命令：

```bash
git init #初始化项目
git remote add origin  git@gitcafe.com:username/username #添加远程仓库地址，origin 这个名字可自定义，指代后面的远程仓库
git checkout -b gitcafe-pages #创建并切换到 gitcafe-pages branch
```
	
3.在该文件夹下用 Jekyll 搭好博客，然后执行 `add` 和 `commit`，就可以推送到远程仓库了：

```bash	
git push origin gitcafe-pages
```

到 Gitcafe 网站上，应该就能看到刚推送的所有文件。

其后的网站更新，依此 `push` 即可。

对于 Github 呢，依样画葫芦：

```bash	
cd  ~/.ssh #进入目录
ssh-keygen -t dsa -C "YOUR_EMAIL@YOUREMAIL.COM" #创建密钥
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): #直接回车
Enter same passphrase again: #直接回车
Your identification has been saved in ~/.ssh/id_dsa.
Your public key has been saved in ~/.ssh/id_dsa.pub.
The key fingerprint is:
15:81:d2:7a:c6:6c:0f:ec:b0:b6:d4:18:b8:d1:41:48 YOUR_EMAIL@YOUREMAIL.COM
cat id_dsa.pub
# 到 Github 网站个人设置项里添加 SSH 公钥，随后测试是否能连接到 Github
# 在 Github 网站新建 username.github.io 项目
# cd 进入本地项目文件夹
git remote add ghorigin git@github.com:username/username.github.io.git #添加远程仓库地址，ghorigin 指代 Github 远程仓库，区别于 Gitcafe 仓库的 origin
```
	
这里需要注意的，除了使用 `ghorigin` 以与 `origin` 区分不同的远程仓库地址之外，还有创建密钥时存放密钥的文件也要和 Gitcafe 的不同。根据 Github [官方文档](https://help.github.com/articles/generating-ssh-keys/)，默认存放公钥的文件有四个：`id_dsa.pub` `id_ecdsa.pub` `id_ed25519.pub` `id_rsa.pub`。既然 `id_rsa.pub` 已经被 Gitcafe 占用，所以这里用了`id_dsa.pub`。

对 Github 执行 `push` 的时候，要选择 `master` branch:

```bash
git push ghorigin master
```

前面说到， github.io 和 gitcafe 使用不同的 branch，会添点小麻烦。因为平时我们在本地编辑博客时，一般只在一个 branch 下工作。假定日常工作在 `gitcafe-pages` branch，每次要 `push` 时，还需要将更新 merge 到 `master` branch。总结起来，需要执行以下命令： 
	
```bash
git push origin gitcafe-pages #先推送到 Gitcafe
git checkout master #切换到本地 master branch
git merge --no-ff gitcafe-pages #将 gitcafe-pages 的更新内容 merge 到 master branch 
git push ghorigin master #推送到 Github
git checkout gitcafe-pages #切换回日常用的 gitcafe-pages branch
```
	
以上还省略了 `add` 和 `commit` 命令，因为我个人是通过 Visual Studio Code 完成这部分操作的。建议把这几个命令写成脚本，每次 `push` 执行脚本即可，更方便。

2015-11-25 更新： 经测试，同一个公钥可以用于不同的远程代码仓库。所以同时维护 Github 和 Gitcafe，只使用 rsa 足矣。

### 域名解析

如果你还注册了个人域名，则要使用域名解析服务，将你的域名指向 Github 和 Gitcafe。这里推荐使用 dnspod 的域名解析服务，免费，同时支持根据访问 IP
来源设定指向不同的 IP 地址。可设定如果访问来自国外则指向 Github，来自国内则指向 Gitcafe，优化访问速度。比如我的设定：

![lLuiX6.png](https://s2.ax1x.com/2020/01/14/lLuiX6.png)
<!-- ![dnspod](/images/dnspod.png) -->

第一条指向 Github，第二条指向 Gitcafe。 

最后在 Github 的项目目录下添加名为 `CNAME` 的文件，里面写上你的个人域名。这样当访问 username.github.io 的时候，Github 会自动跳转你的个人域名。Gitcafe 则直接在项目设置的 Pages 服务里添加个人域名即可。

PS: 根据 Github [官方文档](https://help.github.com/articles/tips-for-configuring-an-a-record-with-your-dns-provider/)，在 DNS 解析设置时，要添加一个 A 记录，指向 Github 的 IP 地址 192.30.252.153。实际上这种方法并不理想。因为大型网站的 IP 地址常常更换，虽然旧的 IP 可能也能用，但访问速度会非常慢。比如我在香港，现在 ping 上面的	192.30.252.153，需要约 200 ms。所以更好的方法是像 Gitcafe 一样，设置一个 CNAME 记录，指向 github.io，如上面截图所示。这样访问的 IP 地址会根据 Github 服务的默认 IP 地址实时更新。目前 ping 一下 github.io 和 gitcafe.io，用时都在 5ms 左右。

2016-03-14 更新： GitCafe 官方通知说 GitCafe 即将停止服务。因为发现 Github 在大陆又可以访问，所以直接删除了在 GitCafe 的项目，一律使用 Github 了。 