---
title: "Mac OS 下打造 golang nvim 编程环境之基础配置"
date: 2022-03-29T22:09:13+08:00
tags: ["总结"]
categories: ["vim","golang"]

---

# 目的

为了提升编程效率，本着少即是多的原则，开始打造自己的精兵利器。

先看效果

![image-20220329221657580](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2022-03-29-141658.png)

使用 vim 的好处就是不需要再依赖于鼠标或者妙控板之类的外设，能专注于编码工作，所见即所得。

# 依赖环境

对于本次配置，基本上采用的都是就目前来说最新的版本，如下 checklist：

- Golang v1.18
- iterm2
- Oh-my-zsh
- brew
- Vim 
- NeoVim
- Ctags
- vim-plug
- vim-go 
- coc-go 
- Hack Nerd Font

为了不受网络影响还需要一个全局代理，也就是说会科学上网。

# 安装步骤

## 安装 golang 

golang 安装教程自行 Google，下载完成后使用 `go env` 进行验证。

goenv 中 `GOROOT` 代表的是 golang 安装的位置，mac 上默认的是 ` /usr/local/go` 位置，如果你自定义安装了，可以使用

```go
go env -w GOROOT=安装位置
```

`GOPATH` 是指 golang 下载的二进制可执行文件或者三方包存放的位置，mac 默认是 ` ${HOME}/go `

## 安装字体

iterm2 + oh-my-zsh 的配置自行 Google，根据自己喜好自定义主题样式。

由于 vim 中图标是 non-ascii 字体，直接使用会出现 ？的乱码，所以需要安装 `Hack Nerd Font` 字体，使用如下命令

```shell
brew tap homebrew/cask-fonts
brew cask install font-hack-nerd-font
```

安装完后进入 iterm2-> Performance -> Profiles -> Text 选择 `Hack Nerd Font`

![image-20220329224414336](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2022-03-29-144414.png)

## 安装 vim 插件

Neovim 安装配置、vim-plug 的使用请自行 Google。

本次涉及的 vim 插件

``` bash
 Plug 'scrooloose/nerdtree', { 'on':  'NERDTreeToggle' }
 Plug 'morhetz/gruvbox'
 Plug 'vim-airline/vim-airline'
 Plug 'vim-airline/vim-airline-themes'
 Plug 'mhinz/vim-startify'
 Plug 'Xuyuanp/nerdtree-git-plugin'
 Plug 'ryanoasis/vim-devicons'
 Plug 'airblade/vim-gitgutter'
 Plug 'fatih/vim-go'
 Plug 'neoclide/coc.nvim', {'branch': 'release'}
 Plug 'preservim/tagbar'
```

使用 `:PlugInstall` 进行安装，等待下载完成，然后进行依赖安装

### 下载 vim-go 依赖

在 vim 中使用 `:GoInstallBinaries` 命令将 vim-go 所需的依赖下载到本地

### 下载 coc-go

在 vim 中使用 `:CocInstall coc-go ` 命令安装 coc 对 go 语言的支持

### 配置 coc

在 coc-go 安装完后需要对其进行配置，核心依赖是 gopls

在 vim 中使用 `:CocConfig` 命令打开配置文件，将 golang 配置粘贴进去

```json
{"languageserver": {
    "golang": {
      "command": "gopls",
      "rootPatterns": ["go.work", "go.mod", ".vim/", ".git/", ".hg/"],
      "filetypes": ["go"],
      "initializationOptions": {
        "usePlaceholders": true
      }
    }
  }
  }
```

额外的配置可以参考 [coc-go](https://github.com/josa42/coc-go) 和 [coc.nvim](https://github.com/neoclide/coc.nvim)

## 更新 ctags

在使用 tagbar 插件时，会依赖系统的 ctags，如果在开启 tagbar 时出现如下错误，那么就需要更新系统的 ctags： 

```bash
Tagbar: Ctags doesn't seem to be Exuberant Ctags!
```

更新方式：

```bash
brew install ctags
```

# 测试

在安装完后，我们可以新建一个 go 项目进行测试

``` bash
$ mkdir test
$ cd test
$ go mod init example.com/test
$ vi main.go
```

首次编辑 go 语言文件时，coc-go 会初始化环境，并且下载 gopls 到 `${HOME}/.config/coc/extensions/coc-go-data/bin` 中，如果顺利的话你的界面将会是下面这个样子

![image-20220329231857748](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2022-03-29-151858.png)

此时我们使用 `:TagbarToggle` 和 `:NERDTreeToggle` 分别将文件结构和目录结构打开，也就是如下界面

![image-20220329232039274](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2022-03-29-152039.png)

此时我们的基础配置就算完成了，后续在使用中不断优化打磨配置文件，最终变成一款得心应手的编辑器。

附：[本人 nvim 目前配置](https://gist.github.com/AhianZhang/2300da5d31c0d8d00b353486f07e9071)

# 总结

在 go v1.18 中下载依赖时使用 `go install` 而不是之前的 `go get` 命令，后面在会再出一篇关于提升使用体验的文章。

# 参考文章

https://github.com/fatih/vim-go

https://github.com/BroQiang/vim-go-ide

https://pkg.go.dev/golang.org/x/tools/gopls#section-readme





