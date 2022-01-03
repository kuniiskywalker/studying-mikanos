# 第一章 PCの仕組みとハローワールド

## Install VSCode plugin Hex Editor(バイナリエディタ)インストール

VSCodeプラグインHex Editorのインストール

バイナリ確認用

## エミュレーターインストール

```
brew install qemu
```

## dosfstoolsインストール

macでmkfs.fat使うために必要

```
brew install dosfstools
```

###　PATHに追加

```
vi ~/.zshrc

export PATH=/usr/local/Cellar/dosfstools/4.2/sbin:$PATH
```

## バイナリからQEMUで起動

### バイナリファイルをイメージに書き込み

```
$ qemu-img create -f raw disk.img 200M
$ mkfs.fat -n 'MIKAN OS' -s 2 -f 2 -R 32 -F 32 disk.img
$ hdiutil attach -mountpoint mnt disk.img
$ mkdir -p mnt/EFI/BOOT
$ curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/day01/bin/hello.efi
$ cp hello.efi mnt/EFI/BOOT/BOOTX64.EFI
$ hdiutil detach mnt
```

### エミュレーター起動

UEFI ファームウェア

```
$ curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/devenv/OVMF_CODE.fd
```

NVRAM 

```
$ curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/devenv/OVMF_VARS.fd
```

UEFIブート

```
$ qemu-system-x86_64 -drive if=pflash,file=OVMF_CODE.fd -drive if=pflash,file=OVMF_VARS.fd -hda disk.img
```
