---
{"dg-publish":true,"permalink":"/Work/Linux/ZSH-终端美化/","title":"ZSH-终端美化","tags":["flashcards"],"noteIcon":"","created":"2025-01-21T12:57:27.000+08:00","updated":"2026-03-25T19:04:05.780+08:00","dg-note-properties":{"title":"ZSH-终端美化","tags":["flashcards"],"reference linking":null}}
---


[iTerm2安装配置使用指南——保姆级 - 知乎](https://zhuanlan.zhihu.com/p/550022490)
[这篇 iTerm2 + Oh My Zsh 教程手把手让你成为这条街最靓的仔 - 知乎](https://zhuanlan.zhihu.com/p/145437836)

# 安装oh-my-zsh
```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
## 设置默认 Shell
安装完成后我们需要把zsh设为默认的Shell，下面提供几个常用的命令，供大家参考：
```bash
#查看系统安装的所有Shell
cat /etc/Shells
#查看当前使用的Shell
echo $Shell
#修改默认Shell为zsh
chsh -s /bin/zsh 
```
# 安装插件
## **声明高亮插件zsh-syntax-highlighting**
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```
## **自动填充建议插件zsh-autosuggestions**
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
```
## **快速跳转插件autojump**
```shell
# 安装
git clone https://github.com/wting/autojump.git && cd autojump && ./install.py
# 卸载
./uninstall.py
```
## 安装成功后，进行如下操作:
```bash
#编辑配置文件
vim ~/.zshrc

#找到plugins配置，在括号内增加zsh-autosuggestions,与其他插件之间使用空格分隔开
plugins=(git zsh-autosuggestions zsh-syntax-highlighting autojump)
#添加如下代码
[[ -s /root/.autojump/etc/profile.d/autojump.sh ]] && source /root/.autojump/etc/profile.d/autojump.sh
autoload -U compinit && compinit -u
#退出编辑后执行使配置生效
source ~/.zshrc
```
# 主题
## Powerlevel10k
### 安装
```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k

# 然后需要打开 `~/.zshrc` 设置 `ZSH_THEME`:
ZSH_THEME="powerlevel10k/powerlevel10k"
```
### 重新配置主体
```shell
p10k configure
```
###  卸载
```shell
rm -rf ~/.p10k.zsh
# 删除 ~/.zshrc 文件中 10k 关键词的代码
```
# 字体
[hack.zip](http://10.0.0.5:5000/sharing/2nrceWUTC)
```shell
## 使mkfontscale和mkfontdir命令正常运行
yum install mkfontscale
## 使fc-cache命令正常运行。如果提示 fc-cache: command not found
yum install fontconfig

# 在 /usr/local/share/fonts下建一个目录，把字体拷贝进来，进入目录执行：
mkfontscale
mkfontdir
fc-cache -fv
```


# CentOS 7 安装高版本的zsh教程
### 背景
生产服务器安装的Linux为[Centos7](https://so.csdn.net/so/search?q=Centos7&spm=1001.2101.3001.7020).9，默认使用yum安装zsh只能到5.0.2版本；想要使用[powerlevel10k](https://github.com/romkatv/powerlevel10k#oh-my-zsh)主题就需要≥5.2版本的zsh。因此手动编译安装一下5.8版本的zsh
```shell
# 查看centos版本
$ cat  /etc/redhat-release
# CentOS Linux release 7.9.2009 (Core)

# 使用yum安装
$ yum install zsh

# 查看版本信息
$ zsh --version
# zsh 5.0.2 (x86_64-redhat-linux-gnu)

```
### 手动安装
#### 卸载低版本
```shell
# 卸载当前版本
$ sudo yum remove zsh

```
#### 下载源码
- 源码：https://zsh.sourceforge.io/Arc/source.html
```shell
# 下载
$ wget https://jaist.dl.sourceforge.net/project/zsh/zsh/5.8/zsh-5.8.tar.xz

# 解压
$ tar xvf zsh-5.8.tar.xz
```
#### 编译安装
```shell
# 安装编译工具
$ yum install gcc perl-ExtUtils-MakeMaker
$ yum install ncurses-devel

# 进入源码目录
$ cd zsh-5.8

# 执行配置
$ ./configure   # 默认安装在：/usr/local/bin/zsh

# 编译和安装
$ make && make install

# 添加信息
$ vim /etc/shells
# 在最后一行加上：/usr/local/bin/zsh
```
#### 验证结果
```shell
# 切换shell
$ chsh -s /usr/local/bin/zsh

# 查看版本信息
$ zsh --version
# zsh 5.8 (x86_64-pc-linux-gnu)
```