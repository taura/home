
# インストール中

## Wi-Fiつなげる

```
iwctl
station wlan0 connect [ESSID]
```

* 参考: https://www.youtube.com/watch?v=3czrHtFHteY

## archinstall 中

* 以下はインストール中に additional package で指定して入れておく
  * Wi-Fi (iwctl) : iwd
  * 何か編集する : nano
  * Bluetooth : bluez bluez-utils networkmanager
  * Desktop : plasma kde-applications sddm 

* 本当はdesktopは後から入れたいところではある
* しかしそれをいれるのがWi-Fi networkにつながった環境を作るためのてっとり早い方法

* iwd について詳細は https://wiki.archlinux.org/title/Iwd
* iwd を入れて起動後に iwctl を使えば network につながるところまでは行く (以下の設定を作る必要があった. そうしないとIP addressが取れない)
```
# /etc/iwd/main.conf

[General]
EnableNetworkConfiguration=True
```
* だがネットにつながっても, DNS 名前解決ができない
  * bind を入れておけばいいのかなとも思うのだが最終的に作りたい環境がそうなのか自信がない
  * 上記ページ `Select DNS manager` の節を見る限り何もしなくてもできても良さそうな雰囲気であるが...

# インストール後

* 最初はCUIログイン
* GUIログインにしたければ
```
sudo systemctl enable sddm
```

* 以下をやらないとbluetoothは使えないっぽい
```
sudo systemctl enable bluetooth
```

# encrypted home のマウント

## /etc/crypttab
```
enc_drv PARTLABEL=enc_part none
```

## /etc/fstab
```
LABEL=enc_home	/home	ext4	defaults,errors=remount-ro,noatime,nofail,x-systemd.device-timeout=10s	0	1
```

# 必須アプリのインストール

```
sudo pacman -S emacs firefox 
```

# yay のインストール

```
sudo pacman -S --needed base-devel git
cd /tmp
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

# yay でいれるアプリのインストール

```
yay -S slack-desktop zoom dropbox
```

# フォント

```
yay -S otf-takao
```

設定は以下を `~/.config/fontconfig/fonts.conf` に書き込む感じ

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>TakaoPGothic</family>  <!-- または IPA明朝なら IPAMincho -->
      <family>Noto Sans CJK JP</family> <!-- fallbackとして残す -->
    </prefer>
  </alias>

  <alias>
    <family>serif</family>
    <prefer>
      <family>TakaoMincho</family>
      <family>Noto Serif CJK JP</family>
    </prefer>
  </alias>

  <alias>
    <family>monospace</family>
    <prefer>
      <family>TakaoGothic</family>
      <family>Noto Sans Mono CJK JP</family>
    </prefer>
  </alias>
</fontconfig>
```

関連コマンド

```
fc-list
fc-match
```

# 日本語設定

```
sudo pacman -S fcitx5 fcitx5-configtool fcitx5-mozc fcitx5-gtk fcitx5-qt
```

以下の設定を `~/.config/environment.d/なんとか.conf` に書いておく (実際は `~/.profile` に書いて, `~/.config/environment.d/env.conf` -> `~/.profile` というリンクを作る (下記参照)

```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS="@im=fcitx"
```

# 仮想マシン関係

```
sudo pacman -Q qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils libvirt
```

* `virt-manager` だけではダメみたい (Virtual Machine Manager の画面で connection が表示されない)

* 起動する前に以下 `/var/lib/libvirt` を -> `/home/tau/vms/libvirt` へのリンクにする
* (必要なのは images の中だけなので慎重を期すなら
  * `/home/tau/vms/libvirt` をrename `/home/tau/vms/libvirt_x`
  *  `/var/lib/libvirt` を `/home/tau/vms/` に cp
  *  `/home/tau/vms/images` (空のはず) を rmdir
  *  `/home/tau/vms/libvirt_x/images` を `/home/tau/vms/libvirt/` に mv
  *  `/var/lib/libvirt` を `/var/lib/libvirt_org` か何かに rename
  *  `/var/lib/libvirt` -> `/home/tau/vms/libvirt` に ln -s)
    
```
sudo systemctl enable libvirtd
```

```
sudo usermod -aG libvirt,kvm $USER
```

* 再起動
* 再起動後に無事 virt-manager で QEMU/KVM という connection がちゃんと有効になり, 新しい仮想マシンを作れるようになるはず
* 上記 (`/var/lib/libvirt` を -> `/home/tau/vms/libvirt` へのリンクにする) をやったおかげで existing image を選んで仮想マシンを作るところで昔作った qcow2 ファイルが表示されるはず

# 設定について今回わかったこと

* GUI (sddm) 環境は 
  * `~/.config/environment.d/` 下の, `*.conf` ファイルを読む
  * `~/.profile` や ましてや `~/.bash_profile` も `~/.bashrc` は読まない
* terminal の shell から起動するのであれば `~/.bash_profile` に書いた設定が継承されるが, 直接起動する GUI アプリ (emacs の M-x compile なども含む) の場合, `~/.bash_profile` の設定が読まれないので不都合
* `~/.config/environment.d/env.conf` -> `~/.profile` というリンクを作って解決

# .bashrc と .bash_profile

* alias や関数などは子シェル引き継がれない
* umask, 環境変数は子シェル (プロセス) に引き継がれる (`PS1` は例外)

ので, 一般論としては `~/.bash_profile` にumask, 環境変数の設定を書き, シェル固有の alias や 関数は `~/.bashrc` に書くらしい

* 以前, 面倒なので, 全部 `~/.bashrc` に書いて, `~/.bash_profile` はそれを source していたがこのたび, 分離した
* そして `~/.config/environment.d/env.conf` から `~/.profile` をリンク
  * 実際は自分は `~/.profile` は作らずに `~/.bash_profile` を使っており祖霊自体が `~/cfg3/rc/_bash_profile` というファイルへのリンクなので `~/.config/environment.d/env.conf` -> `~/cfg3/rc/_bash_profile` というリンク

# Python virtual env と virt-manager

* 最近のUbuntuで pip を使いたければ virtual env というノリになってきたので `~/.bash_profile` 中で virtual env を activate していたがそれをやると virt-manager がそれを使ってしまい, その中に `gi` というモジュールがない, というトラブルを引き起こした
* なので `~/.bash_profile` 中で virtual env を activate するのをやめた
* ただしそうすると, GUI アプリ (emacs 内の Python 環境を含む) で virtual env を使えないという不便がある
