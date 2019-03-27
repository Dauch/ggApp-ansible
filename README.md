landload-ansible
=============
landload-ansible是[landload](https://github.com/sundream/ggApp)的ansible配置示例,可以利用[ansible](https://github.com/ansible/ansible)快速部署服务器

Table of Contents
================

* [名字](#landload-ansible)
* [已测试环境](#已测试环境)
* [安装ansible](#安装ansible)
* [部署landload](#部署)
* [管理landload](#管理)
* [打包和发布](#打包和发布)


已测试环境
=========
* Ubuntu-18.04.1
* Ubuntu-16.04.5
* Centos-7.5

安装ansible
===========
* [installation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
* ubuntu
```
sudo apt install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
# 使用pip安装
sudo apt install python
sudo apt install python-pip
pip install ansible
```
* centos
```
# 安装pip(用于安装ansible)
sudo easy_install pip
# 使用pip安装
sudo pip install ansible
#pip install git+https://github.com/ansible/ansible.git@devel
```

[Back to TOC](#table-of-contents)

部署
====
```
# 暂时不支持macos下自动部署,ubuntu18.14.1/centos下测试通过
# 免密登录
ssh-keygen   #一路按回车跳过
ssh-copy-id -i ~/.ssh/id_rsa.pub $USER@127.0.0.1  #中途需要输入用户密码
cd ~ && git clone https://github.com/Dauch/ggApp-ansible
cd ~/landload-ansible
# 提前安装所有依赖软件/库
ansible-playbook -i hosts/gamesrv.test --limit gamesrv_1 install.yml -e home=$HOME -K
# 部署accountcenter
ansible-playbook -i hosts/accountcenter.test --limit accountcenter deploy_accountcenter.yml -e home=$HOME -K
# 部署gamesrv
ansible-playbook -i hosts/gamesrv.test --limit gamesrv_1 deploy_gamesrv.yml -e home=$HOME -K
```
部署完后会在本机生成如下目录
```
//landload工作目录
~/landload
	+accountcenter		//账号中心
	+gamesrv			//游戏服
	+client				//简易客户端
	+robot				//机器人压测工具
	+tools				//其他工具

//依赖软件源码目录
/usr/local/src
	+lua-5.3.5
	+openresty-1.13.6.2
	+luarocks-3.0.4

//依赖软件二进制包目录
/usr/local
	+openresty
	+bin
		+lua
		+luarocks
		+luarocks-5.3

# 可以执行以下指令检查安装软件的版本
lua -v
openresty -v
luarocks --version
```

[Back to TOC](#table-of-contents)

管理
====
* 管理accountcenter
```
# 启动
ansible accountcenter -i hosts/accountcenter.test -m shell -a "cd ~/landload/accountcenter && /usr/local/openresty/bin/openresty -c conf/account.conf -p . &"
# 导入游戏服务器信息
ansible accountcenter -i hosts/accountcenter.test -m shell -a "cd ~/landload/tools/script && python import_servers.py --appid=appid --config=servers.config"
# 关闭
ansible accountcenter -i hosts/accountcenter.test -m shell -a "cd ~/landload/accountcenter && /usr/local/openresty/bin/openresty -c conf/account.conf -p . -s stop"
# 重新加载
ansible accountcenter -i hosts/accountcenter.test -m shell -a "cd ~/landload/accountcenter && /usr/local/openresty/bin/openresty -c conf/account.conf -p . -s reload"
```
* 管理gamesrv
```
# 启动
ansible gamesrv_1 -i hosts/gamesrv.test -m shell -a "cd ~/landload/{{inventory_hostname}}/shell && sh start.sh"
# 关闭
ansible gamesrv_1 -i hosts/gamesrv.test -m shell -a "cd ~/landload/{{inventory_hostname}}/shell && sh stop.sh"
# 重新启动
ansible gamesrv_1 -i hosts/gamesrv.test -m shell -a "cd ~/landload/{{inventory_hostname}}/shell && sh restart.sh"
# 强制关闭(非安全关闭)
ansible gamesrv_1 -i hosts/gamesrv.test -m shell -a "cd ~/landload/{{inventory_hostname}}/shell && sh kill.sh"
# 查看启动状态
ansible gamesrv_1 -i hosts/gamesrv.test -m shell -a "cd ~/landload/{{inventory_hostname}}/shell && sh status.sh"
# 执行gm
ansible gamesrv_1 -i hosts/gamesrv.test -m shell -a "cd ~/landload/{{inventory_hostname}}/shell && sh gm.sh 0 exec 'return 1+1'"
```

打包和发布
==========
```
# 打包前先更新代码！！！
# 打整包
# 查看帮助
sh shell/pack.sh
# 对账号中心打整包
sh shell/pack.sh ~/landload/accountcenter
# 执行shell/pack.sh后会提示生成的包名
# 发布到账号中心
ansible-playbook -i hosts/accountcenter.test --limit accountcenter publish.yml -e packname=包名

# 对游戏服打整包
sh shell/pack.sh ~/landload/gamesrv
# 执行shell/pack.sh后会提示生成的包名
# 发布到所有游戏服
ansible-playbook -i hosts/gamesrv.test --limit gamesrv publish.yml -e packname=包名
# 发布到gamesrv_50,gamesrv_51服
ansible-playbook -i hosts/gamesrv.test --limit gamesrv_50,gamesrv_51 publish.yml -e packname=包名

# 打补丁包
# 查看帮助
sh shell/packpatch.sh
# 仓库是git管理
# 对游戏服最近2次提交生成补丁包
sh shell/packpatch.sh ~/landload/gamesrv HEAD~2..HEAD
# 对账号中心最近2次提交生成补丁包
sh shell/packpatch.sh ~/landload/accountcenter HEAD~2..HEAD
# 仓库是svn管理
# 对游戏服[530,532]之间提交生成补丁包
sh shell/packpatch.sh -s ~/landload/gamesrv 530:532
# 对账号中心[530,532]之间提交生成补丁包
sh shell/packpatch.sh -s ~/landload/accountcenter 530:532
# 执行shell/packpatch.sh后会提示生成的补丁包名
# 向游戏服发布补丁包并自动热更
ansible-playbook -i hosts/gamesrv.test --limit gamesrv publish.yml -e hotfix=true -e packname=补丁包名
# 向账号中心发布补丁包
ansible-playbook -i hosts/accountcenter.test --limit accountcenter publish.yml -e packname=补丁包名 
# 让账号中心热更
ansible accountcenter -i hosts/accountcenter.test -m shell -a "cd ~/landload/accountcenter && /usr/local/openresty/bin/openresty -c conf/account.conf -p . -s reload"
```

[Back to TOC](#table-of-contents)
