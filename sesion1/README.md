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

### PATHに追加

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

#### UEFI ファームウェア

```
$ curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/devenv/OVMF_CODE.fd
```

#### NVRAM 

```
$ curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/devenv/OVMF_VARS.fd
```

#### UEFIブート

```
$ qemu-system-x86_64 -drive if=pflash,file=OVMF_CODE.fd -drive if=pflash,file=OVMF_VARS.fd -hda disk.img
```

## memo: UEFIとBIOSの違い

### UEFI

1.もっと直感的なインターフェイス（マウスでも操作可能）
2.ハードディスクの容量が2.2TBを超えても対応可能
3.スタートアップトシャットダウンをスピードアップ
4.遠隔診断や起動前の検査などのセキュリティ機能が追加
5.ネットワーク機能が追加

#### UEFIブートプロセス

```
UEFI -> GPT（EFIブートローダー） -> Kernel -> OS
```

### BIOS

1. BIOSは、16ビットモードにしか対応してない
2. アドレス空間も1MBのみ
3. 同時に複数のハードウェアを制御することができないため、パソコンが起動するまでとても時間がかかる
4. 2TB以上のディスクをサポートできない

#### BIOSブートプロセス

```
BIOS -> MBR -> ブートローダー -> Kernel -> OS
```

## Cで書かれたEFIプログラムをコンパイルしてQEMUで起動

### コンパイラとリンカの準備

clangは既に入っていた（xcodeインストール時になんかにインストールされたっぽい）

```
brew install llvm nasm
```

### PATHに追加

```
vi ~/.zshrc

export PATH=/usr/local/Cellar/llvm/12.0.1/bin:$PATH
```

### サンプルのCのソースダウンロード

```
$ curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/day01/c/hello.c
```

### Cのソースコンパイル

```
$ clang -target x86_64-pc-win32-coff -mno-red-zone -fno-stack-protector -fshort-wchar -Wall -c hello.c
```

### コンパイルしたものをリンク

```
$ lld-link /subsystem:efi_application /entry:EfiMain /out:hello.efi hello.o
```

### QEMUの起動

「バイナリからQEMUで起動」同様のやり方で起動

以下方法で短縮可能

```
$ cd && git clone -b karaage https://github.com/karaage0703/mikanos-build osbook
```

```
$ ~/osbook/devenv/run_qemu.sh hello.efi
```