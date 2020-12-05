---
title: Homebrew更新相关小技巧
date: 2020-11-26 15:22:22
tags: [OS X & macOS, Homebrew, Software]
thumbnail: https://brew.sh/assets/img/homebrew-256x256.png
---

[原文链接](https://blog.chaosjohn.com/Homebrew-upgrade.html)
# 前言
笔者在四年前曾写过一文[HomeBrew -- OSX下的最强软件包管理器](/2016/05/26/HomeBrew-The-Best-Package-Manager-On-OSX/)，篇中详细介绍了HomeBrew以及Cask的使用。

多年过去了，Homebrew依旧非常流行，但内在也发生了很多变化，比如Cask原先作为第三方Tap的存在，现已整合到Homebrew内，在未安装Cask的环境里，执行 `brew cask install ${app}`，Homebrew则会自动安装Cask。

同时，在Homebrew的日常使用中，笔者也遇到了很多问题，本文就着重于“踩坑”和“填坑”。

# “这些年遇到的坑”
## Homebrew自动更新
*brew* 提供了 *update* 命令，但是你执行 *install* 或 *upgrade* 时，都会强制性先 *update*，浪费宝贵的时间。

**解决**：
- brew命令前加上变量HOMEBREW_NO_AUTO_UPDATE=1，变为 `$ HOMEBREW_NO_AUTO_UPDATE=1 brew install …`
- 或使用alias别名，在.bashrc或.zshrc中新增一行 `alias brew="HOMEBREW_NO_AUTO_UPDATE=1 brew"`
- 或导出环境变量，在.bashrc或.zshrc中新增一行 `export HOMEBREW_NO_AUTO_UPDATE=1`，笔者推荐这种方法
- 或者使用 *Homebrew/aliases*，执行 `brew alias install_no_autoupdate='!HOMEBREW_NO_AUTO_UPDATE=1 brew install'`（*install_no_autoupdate* 名字任意更换），以后要执行无预更新的 `brew install ${formula}` 操作，都改为 `brew install_no_autoupdate ${formula}`

## **Cask** 安装的应用，和 **App Store** 安装的应用，都混在了一起
因为双方都把应用安装到了 */Applications* 目录下，所以导致
1. 分不清应用到底是Cask安装的，还是App Store里安装的
2. 如果Cask和App Store安装同一款应用，则后安装或更新的应用，就会覆盖掉先前存在的版本

**解决**：`export HOMEBREW_CASK_OPTS="--appdir=~/Applications/_"`，这样Cask会自动将应用都安装到用户目录下的 `Applications/_/` 里。

这里我想介绍一下我在mac下的应用管理，仅供参考：
- App Store安装的应用，都位于 `/Applications/` 下
- Cask安装的应用，都位于 `~/Applications/_/` 下
- 网络上搜罗来的破解应用，都位于 `~/Applications/#/` 下
- JetBrains公司的IDE，都用 `JetBrains Toolbox` 进行安装管理（当然Toolbox本身是用Cask进行安装的）
- 其他开源/免费的应用，都位于 `~/Applications/` 下

## **Cask** 批量更新应用
在前文[HomeBrew -- OSX下的最强软件包管理器](/2016/05/26/HomeBrew-The-Best-Package-Manager-On-OSX/)中笔者曾给出过一行shell脚本用来批量更新Cask的应用，但是这么多年过去了，这行脚本笔者不再推荐使用。
**解决**：`brew tap buo/cask-upgrade`，[项目链接](https://github.com/buo/homebrew-cask-upgrade)，安装完之后，
- 执行: `brew cu`，更新所有“存在更新版本”的应用
- 执行: `brew cu ${app}`，更新特定app
- 选项: `-a, --all`，包含标记了 *auto-update* 的应用；`-f, --force`，包含当前版本号为 *latest* 的应用；`-y, --yes`，对所有询问是否确认更新，自动应答 *yes*


## 因国内网络环境导致brew速度慢
这里的慢包含两方面：`brew update` 慢 & `brew install` 慢

**解决方案A**: 换源策略
1. `brew update` 慢，[参考清华镜像源指南](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/)
2. `brew install` 慢，[参考清华镜像源指南](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew-bottles/)

**解决方案B**: 代理策略, `export all_proxy=socks5://${host}:${port}`，替换自己代理主机地址和端口即可


## Error: SHA256 mismatch / Error: Checksum mismatch
前者是 `brew` 后者是 `cask`，错误原因，仓库里记载的校验值和实际下载下载的文件校验值不一致

**解决**: 执行 `rm -rf ~/Library/Caches/Homebrew` 将本地缓存目录删除后重试，有概率能解决问题，适用于软件发行者修改应用后缺没有更改版本号就发行出去的情况。若未解决，针对于 `brew`，编辑 `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/${formula}.rb`，针对于 `cask`，编辑 `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask/Casks/${app}.rb`，将 `sha256`修改为实际校验值，保存，再次执行先前操作，大概率能成功。切记两点主意事项：
1. 需禁用brew的自动更新，否则前脚刚改完校验值，后脚自动更新就把校验值改回来了
2. 更改校验值后不要关闭编辑器，等安装/更新应用成功后，立即撤销更改，再关闭编辑器，因为brew和cask的仓库是用git管理的，更改仓库文件对后续的更新会造成冲突。如果发生冲突了，也别担心，到仓库根目录执行 `git reset --hard`恢复原样。*brew* 的仓库目录位于 `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/`；*cask* 的仓库目录位于 `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask/`

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)