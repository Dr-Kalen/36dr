title: Ubantu搭建Git服务器
date: 2016-01-06 00:52:53
tags: git gitlite gitosis gitweb
---
### Ubantu部署Git服务器基本配置环境
#### 第一步，检查ssh是否正常
检查ubantu服务器是否安装过ssh-server，Ubuntu默认并没有安装ssh服务，如果通过ssh链接Ubuntu，需要自己手动安装openssh-server。判断是否安装ssh服务，可以通过如下命令进行：
``` linux
ssh localhost
The authenticity of host 'localhost (127.0.0.1)' can't be established.
```
<!--more-->
如果服务器未安装ssh-server，当通过ssh连接服务器时会报出
``` text
ssh: connect to host localhost port 22: Connection refused
```
既然没有安装ssh-server，则通过命令安装ssh-server的方法
``` linux
sudo apt-get install openssh-server
```
然后再次使用命令来检验安装是否成功
``` linux
ps -e|grep ssh
or
ssh localhost
```

#### 第二步，创建管理账户，如git
在ubantu服务器上使用命令创建git账户和设置密码
``` linux
$ sudo useradd -m git
$ sudo passwd git
```

#### 第三步，上传管理服务器的信任秘钥
首先需要创建秘钥，若电脑上~/.ssh/目录下已经有id_rsa.pub文件则不用考虑一下命令可忽略。
``` linux
$ ssh-keygen
```
一直回车，会在~/.ssh/目录下生成两个文件：
id_rsa
私钥文件。是基于RSA算法创建。该私钥文件要妥善保管，不要泄。
id_rsa.pub
公钥文件。和id_rsa文件是一对儿，该文件作为公钥文件，可以公开。
然后将已经生成好的id_rsa.pub上传到ubantu服务器上/tmp目录以备使用
``` linux
$ scp  ~/.ssh/id_rsa.pub git@server:/tmp
```
#### 第四步，服务器安装git
在Ubantu服务器上安装git软件，使用命令
``` linux
$ sudo apt-get install git
```
### Gitolite服务器部署方法
Gitolite官方github地址：https://github.com/sitaramc/gitolite
Gitolite官方安装方法：http://gitolite.com/gitolite/install.html
#### 第一步，克隆Gitolite源码
``` linux
// 切换到git用户
su git
// 切换到git根目录
cd
// clone gitolite
$ git clone https://github.com/sitaramc/gitolite.git
```
#### 第二步，安装个Gitolite
进入.ssh目录，如果有authorized_keys，删除即可。
``` linux
// 新建bin目录
mkdir bin

// 安装gitolite，默认会安装到bin，如果想安装到你自己之的指定的目录请参考官方安装 -to
gitolite/install -ln

// 配置秘钥
bin/gitolite setup -pk admin.pub
```
测试是否执行成功
首先git根目录下是否生成了projects.list和repositories
同时可以进入.ssh，可以看到新生成的authorized_keys，以后每一次提交新用户都会写到这个里边。判断是否用户添加成功，看这个里边文件是否新增了那个用户的key即可。

#### 第三步，客户端管理用户
首先使用刚刚提交id_rsa电脑克隆gitolite-admin代码
``` linux
// 结尾不需要加.git
$  git clone git@server:gitolite-admin
```
修改gitolite-admin/conf/gitolite.conf
``` text
@developer  =   kalen bob

repo gitolite-admin
    RW+     =   kalen

repo testing
    RW+     =   @all

repo release
    RW+     =   @developer
```
同时将新用户的key都放到keydir，记住添加的.pub文件名一定要与gitolite.conf中配置的名称一致

push到服务器即可，查看是否成功，去服务器进入git用户，查看.ssh/authorized_keys，里边会多了新的key。

#### 第四步，了解和配置gitolite权限

权限配置在gitolite.conf中进行，注释用#表示。
C
C 代表创建。仅在 通配符版本库 授权时可以使用。用于指定谁可以创建和通配符匹配的版本库。
R, RW, 和 RW+
R 为只读。RW 为读写权限。RW+ 含义为除了具有读写外，还可以对 rewind 的提交强制 PUSH。
RWC, RW+C
只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 C 的含义是允许创建和正则引用匹配的引用（分支或里程碑等）。
RWD, RW+D
只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 D 的含义是允许删除和正则引用匹配的引用（分支或里程碑等）。
RWCD, RW+CD
只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 C 的含义是允许创建和正则引用匹配的引用（分支或里程碑等），D 的含义是允许删除和正则引用匹配的引用（分支或里程碑等）。
-
 是一条禁用指令。只对写操作起作用，即禁用用户的写操作。
接下来实际分析一个稍微复杂一些的配置文件
``` text
@admin = git kalen admin1 admin2
2   @dev = dev1 dev2 dev3 fish
3
4   repo gitolite-admin
5       RW+                 = git kalen
6
7   repo Projects/.+
8       C                   = @admin
9       RW                  = @all
10
11  repo testing
12      RW+                  =   @admin
13      -                    =   fish
14      RW      master       =   @dev
15      RW+     dev          =   dev1
16      RW      wip$         =   dev2
```
逐行解释：
1. @admin用户组有git keven admin1 admin2四个用户
2. @devteam用户组有dev1 dev2 dev3 fish四个用户
4. 对于gitolite-admin仓储
5. git keven两个用户拥有读/写/强制更新的权限
7. 对于Projects下所有的git仓储（/.+代表递归所有）
8. @admin用户组拥有创建仓储的权限
9. 所有人均可读/写
11. 对于testing.git
12. @admin用户组拥有读/写/强制更新的权限
13. fish是新手，对其屏蔽写的权限。因为其属@dev组，则还只剩下R 读的权限
14. @dev用户组对master开头的分支拥有读/写权限
15. dev1这个用户对dev开头的分支拥有读/写/强制更新的权限
16. dev2这个用户对于wip分支（严格匹配）具有读/写权限

#### 高级篇，gitolite创建仓库方法
1. 登录远程服务器创建
ssh登录服务器，切换至git用户，进入相关目录，创建某仓储
``` linux
mkdir somegit.git
cd somegit.git
git init --bare
```
2. 修改gitolite.conf创建仓储
打开gitolite-admin/conf/gitolite.conf，添加：
``` text
repo testing2
    RW+    =  @all
```
保存修改，提交。
``` linux
git@linux-dev:~/gitolite-admin$ git commit-m'add test2'
[master b26be9a] add test2
1 file changed, 4 insertions(+)
git@linux-dev:~/gitolite-admin$ git push origin master
Counting objects: 7, done.
Delta compression using up to2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4),350 bytes, done.
Total 4 (delta 1), reused0 (delta0)
remote: 初始化空的 Git 版本库于 /home/git/repositories/testing2.git/
To git@127.0.0.1:gitolite-admin
   0c409e4..b26be9a  master -> master
```
可以看到，gitolite会自动检测配置文件，发现目前没有的仓储会自动才创建。
3. 高端大气上档次
对于通配符版本库，即repo Projects/.+类型的，在有创建权限的用户shell中，本地执行：
``` linux
mkdir somegit
cd somegit
git init
git commit --allow-empty
git remote add origin git@server:Projects/somegit.git
git push origin master
```
gitolite会直接创建新的仓储。

删除：
1.在conf/gitolite.conf中删除相关仓储配置信息（gitolite不会自动删除服务器上的文件，这点与add不同）；
2.登录服务器删除需要删除的仓储。

重命名
同删除操作的步骤。

以上则是Gitolite的部署方法
* 优点可以对git账户进行权限配置，可以深入到库的分支权限中
* 缺点无法做到像SVN一样分配账户，密码的方式来控制不在rsa的用户进行开发

### Gitosis服务器部署方法
#### 第一步，安装python setuptools
python是gitosis需要的前置条件，所以需要安装
``` linux
$ sudo apt-get install python-setuptools
```
#### 第二步，克隆gitosis源码
``` linux
cd /home/git
git clone git://eagain.net/gitosis.git
cd gitosis
python setup.py install
```

#### 第三步，初始化gitosis和修改可操作权限
``` linux
#切换到服务器
$ sudo -H -u git gitosis-init < /tmp/id_rsa.pub (前面以备用的pub)
#修改post-update权限
$ sudo chmod 755 /home/git/repositories/gitosis-admin.git/hooks/post-update
```
#### 第四步，客户端管理用户
切换到提供id_rsa的电脑，克隆gitosis管理仓库
``` linux
$ git clone git@主机名:gitosis-admin.git
$ cd gitosis-admin
```
修改gitosis.conf文件
``` text
[gitosis]

[group gitosis-admin]
writable = gitosis-admin
members = miao@u32-192-168-1-110
```
group 表示群组
writalbe 表示仓库名称
members 表示可允许操作的用户，用户的名称与keydir上传的pub名称一致

添加keydir和修改gitosis.conf文件之后提交，用户管理完成，即可下载仓库无密码方式提交
``` linux
git commit -am "add member john and project foo"
git push
```
#### gitosis部署常见问题
首先确定 /home/git/repositories/gitosis-admin.git/hooks/post-update 为可执行即属性为 0755
1. git操作需要输入密码
原因
公密未找到
解决
上传id_pub.rsa到keydir并改为'gitosis帐号.pub'形式，如miao.pub。扩展名.pub不可省略
2. ERROR:gitosis.serve.main:Repository read access denied
原因
gitosis.conf中的members与keydir中的用户名不一致，如gitosis中的members = foo@bar，但keydir中的公密名却叫foo.pub
解决
使keydir的名称与gitosis中members所指的名称一致。
改为members = foo 或 公密名称改为foo@bar.pub

引用文献：http://www.dbpoo.com/git-gitolite-gitweb/
