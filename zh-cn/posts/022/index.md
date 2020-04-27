# 用 iTerm2 美化你的 Mac 终端


<!--more-->

## 先看看效果图

![效果图](https://i.loli.net/2019/12/25/bAyehHXWCwkLPSR.png)

## 安装 iTerm2

下载安装 [iterm2](https://www.iterm2.com/)

https://iterm2.com/downloads/stable/latest

安装 [oh-my-zsh](https://ohmyz.sh/)

```c++
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

或者

sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

```c++

```

若需要Homebrew，执行下面命令安装

```c++
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

wget的安装

```c++
brew install wget
```


如需科学上网：[参考](https://www.noxxxx.com/mac-下终端走-ss-代理.html)

在终端中直接运行命令

```c++
export http_proxy=http://proxyAddress:port
```

这个办法的好处是简单直接，并且影响面很小（只对当前终端有效，退出就不行了）。

如果你用的是ss代理，在当前终端运行以下命令，那么`wget` `curl` 这类网络命令都会经过ss代理

```c++
export ALL_PROXY=socks5://127.0.0.1:1080
```


## 下载配色方案

你可以在 [这里](https://iterm2colorschemes.com/) 查看配色方案。（这里我选择 **Tomorrow Night Eighties**）

然后到 [这里](https://github.com/mbadolato/iTerm2-Color-Schemes) 下载 iterm2 的配色方案

下载好之后，在iTerm2首选项中导入配色方案：

![首选项](https://i.loli.net/2019/12/25/b73i1KHnfIRov8N.png)

![选择配色文件](https://i.loli.net/2019/12/25/BTOsYSQXWUZhHV3.png)

## 更换 zsh 默认的主题

我使用的主题是[powerlevel9k](https://github.com/Powerlevel9k/powerlevel9k/)

```c++
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```

编辑配置文件 `vi ~/.zshrc`

```c++
ZSH_THEME="powerlevel9k/powerlevel9k"
# 命令左侧显示内容（当前目录 git 版本信息）
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(context dir vcs)
# 命令右侧显示内容（命令运行状态，剩余内存，时间，cpu负载，剩余电量）
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status ram time load battery)
```

之后运行 `source ~/.zshrc` 命令，让其生效。



## 更换字体

直接使用 `powerlevel9k` 主题的话，因为默认字体不支持 icon，会出现一些乱码。

因为 `powerlevel9k` 中用到了大量的 icon，但是只有部分的字体支持 icon。目前对 icon 的支持得比较好的字体是 [Nerd Font](https://www.nerdfonts.com/#home)。brew 提供了 nerd 字体的安装。

```c++
$ brew tap homebrew/cask-fonts
$ brew search nerd
==> Formulae
container-diff
==> Casks
font-3270-nerd-font
...
# 选择Nerd Font字体进行安装
$ brew cask install font-sourcecodepro-nerd-font
```

安装字体成功后，就可以在 iterm2 中设置了：

![更改字体](https://i.loli.net/2019/12/25/LqtHU8Z95oBdiFD.png)



## 插件

`oh-my-zsh` 有很多好用的插件，安装好的插件需要在 `~/.zshrc` 中的 `plugins=(git autojump cp extract sudo z zsh-syntax-highlighting zsh-autosuggestions)` 依次写入配置，用空格分开。

**autojump**

记忆我们之前去过的目录，不需要多次 `cd` ，直接 `j 目录名` 就可以直接进入。

```c++
#安装
brew install autojump

#在 ~/.zshrc 中加入如下配置
[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh
source $ZSH/oh-my-zsh.sh
```

### **zsh-autosuggestion**

输入命令提示自动补全，然后按键盘 `→` 即可补全

```c++
#安装
$ git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

### **zsh-syntax-highlighting**

命令高亮显示，错误显示红色

```c++
# 安装
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

### **colorls**

 显示文件图标

```c++
# 安装
sudo gem install colorls 
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
