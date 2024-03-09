---
title: Manajaro i3wm 配置
slug: manjaro_i3wm
description: 本文为 Manjaro i3wm 版的配置指南。
image: cover.webp
date: 2023-12-12T18:27:30Z
categories: [Technology]
tags: [manjaro, linux]
---

## 安装系统

{{< notice note >}}

Todo:

- 无人值守安装

{{< /notice >}}

本文为 Manjaro i3wm 版的配置指南，语言环境为 `zh_CN.UTF-8`。

在 Livecd 中启动安装程序前先针对 Livecd 换源和安装字体：

```shell
sudo pacman-mirrors -i -c China -m rank
sudo pacman -Syy
```

## 系统配置

### 基础配置

源与字体显示问题：

```shell
# 换源
sudo pacman-mirrors -i -c China -m rank
# 增加 archlinuxcn 源
sudo echo -e "\n[archlinuxcn]\nServer = https://mirrors.ustc.edu.cn/archlinuxcn/\$arch\n" | sudo tee -a /etc/pacman.conf
sudo pacman -Syy
pacman -Sy archlinuxcn-keyring yay
# 安装字体
yay -Sy noto-fonts noto-fonts-cjk noto-fonts-emoji ttf-dejavu-nerd ttf-sarasa-gothic nerd-fonts-sarasa-term
```

intel 核显加速：

```shell
yay -Sy intel-media-driver libva-utils
```

intel 核显垂直同步：

```shell
sudo echo -e "\nSection "Device"\nIdentifier  "Intel Graphics"\nDriver      "intel"\nOption      "TearFree"    "true"\nEndSection" | sudo tee -a /etc/X11/xorg.conf.d/20-intel.conf
```

hdmi 声音输出问题

```shell
# 获得声卡和通道 id
aplay -l

# 根据上面的情报更改下面的 `${CARD_NUM}` `${DEVICE_NUM}`：
sudo echo -e "\ndefaults.pcm.card ${CARD_NUM}\ndefaults.pcm.device ${DEVICE_NUM}\ndefaults.ctl.card ${CARD_NUM}\n" | sudo tee -a ~/.asoundrc

# 默认为最大音量，可开启一个音频后迅速关掉，通过 alsamixer 调整到合适音量
alsamixer
```

若错误时机箱发出蜂鸣声则可通过设置消除：

```shell
sudo sed -i "s/#set bell-style none/set bell-style none/g" /etc/inputrc
```

设置主文件夹为英文：

{{< notice info >}}

建议将主文件夹设置为英文，避免在命令行中频繁切换输入法。

{{< /notice >}}

```shell
yay xdg-user-dirs-gtk
# 临时改变语言环境变量
export LANG=en_US.UTF-8
# 使用 xdg-user-dirs-gtk 改变配置文件并修改文件夹
# 在弹出的窗口中确认操作
xdg-user-dirs-gtk-update
# 改回语言环境变量
export LANG=zh_CN.UTF-8
# 检查是否正确设置
cat /etc/xdg/user-dirs.defaults
cat ~/.config/user-dirs.dirs
```

{{< notice info >}}

系统基础配置完成后建议重启系统。

{{< /notice >}}

### 基础软件

安装 fcitx5 输入法：

```shell
yay -Sy fcitx5 fcitx5-configtool fcitx5-qt fcitx5-gtk fcitx5-chinese-addons kcm-fcitx5 fcitx5-lua
sudo echo -e "\nGTK_IM_MODULE=fcitx\nQT_IM_MODULE=fcitx\nXMODIFIERS=@im=fcitx\nINPUT_METHOD=fcitx\nSDL_IM_MODULE=fcitx\n" | sudo tee -a /etc/environment
git clone https://github.com/tonyfettes/fcitx5-nord.git
mkdir -p ~/.local/share/fcitx5/themes/
cp -r ./fcitx5-nord/Nord-Dark/ ./fcitx5-nord/Nord-Light/ ~/.local/share/fcitx5/themes/
```

安装 chrome 浏览器：

```shell
yay -Sy google-chrome
# 设置 chrome 为默认应用
sed -i "s/userapp-Pale Moon.desktop/google-chrome-stable.desktop/g" ~/.config/mimeapps.list
sed -i "s/export BROWSER=\/usr\/bin\/palemoon/export BROWSER=\/usr\/bin\/google-chrome-stable/g" ~/.profile
```

其他基础工具和软件：

```shell
sudo pacman -Sy gedit git curl gcc neofetch alacritty fish manjaro-settings-manager lightdm-settings nemo cinnamon-translations nemo-preview nemo-fileroller nitrogen picom rofi file-roller neovim
# 设置默认终端为 alacritty
gsettings set org.cinnamon.desktop.default-applications.terminal exec alacritty
# 设置默认编辑器为 gedit
sed -i "s/mousepad.desktop/org.gnome.gedit.desktop/g" ~/.config/mimeapps.list
# 切换默认 shell
chsh -s /usr/bin/fish
```

完毕后重启，之后的 shell 为 `fish`：

```shell
# 安装 ohmyfish
curl https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install | fish
# fish 使用 ays 主题
omf install ays
omf theme ays

# 切换 fish 配色为 nord
fish_config

# 设置 vim 别名
alias -s vim nvim
alias -s vi nvim
```

主题等：

```shell
git clone https://github.com/TheDistroHopper/i3wm-nord.git
sudo cp -r ./i3wm-nord/themes /usr/share/
sudo cp -r ./i3wm-nord/icons /usr/share/
curl -O https://raw.githubusercontent.com/nordtheme/gedit/develop/src/xml/nord.xml
mkdir -p ~/.local/share/gedit/styles
mv ./nord.xml ~/.local/share/gedit/styles/nord.xml

git clone https://github.com/nekokit/i3wm-dotfiles.git
sudo cp -r ./i3wm-dotfiles/backgrounds /usr/share/
cp -r ./i3wm-dotfiles/.config ~/
cp ./i3wm-dotfiles/.Xresources ~/.Xresources
xrdb -load ~/.Xresources
cp ./i3wm-dotfiles/.dir_colors ~/.dir_colors
```

至此已经安装大部分软件，软件需要经过以下设置：

- `mousepad`：设置配色并取消菜单栏
- `gedit`：设置主题并隐藏菜单栏
- `fcitx5-configtool`：设置输入法皮肤
- `lxappearance`：设置主题、字体、图标
- `nitrogen`：设置桌面背景
- `lightdm-settings`：设置锁屏界面
- `nemo`：设置文件管理器
- `google-chrome-stable`：设置 `Google Chrome Nord Theme` 主题

## 其他软件

### 应用

```shell
yay -Ss jellyfin-media-player sonixd qimgv
# 设置默认看图工具
sed -i "s/viewnior.desktop;gpicview.desktop/qimgv.desktop/g" ~/.config/mimeapps.list
```

### 现代 shell 工具

- `eza`：文件列表，替换 `ls`
- `zoxide`：文件夹跳转，替换 `cd`
- `bat`：文件查看，替换 `cat`
- `hexyl`：16 进制查看器
- `fx`：json 解析
- `fd`：文件查找，替换 `find`
- `fzf`：模糊查找
- `ripgrep`：字符串查找，替换 `grep`
- `duf`：挂载点，替换 `df`
- `dust`：文件占用，替换 `du`
- `dua-cli`：文件管理
- `htop`：列出进程，替换 `top`
- `procs`：列出进程，替换 `ps`
- `dog`：DNS 查询，替换 `dig`
- `sd`：字符串替换，替换 `sed`
- `broot`：列出树状文件夹，替换 `tree`
- `the_silver_searcher`字符串搜索：，替换 `ack`
- `choose`：字符串提取，替换 `cut`
- `gping`：ping 可视化工具，替换 `ping`

```shell
yay -Ss eza zoxide bat hexyl fx fd fzf ripgrep duf dust dua-cli htop procs dog sd broot the_silver_searcher choose gping

alias -s ls "eza"
alias -s ll "eza -l -g -h --time-style long-iso"
alias -s la "eza -a -l -g -h --time-style long-iso"

echo -e "\nzoxide init fish | source\n" | tee -a ~/.config/fish/config.fish

broot
```
