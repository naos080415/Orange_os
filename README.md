# orange_os

# 開発環境の準備
Home brew はすでにインストール済み.
Dockerで環境構築をしても良かったが,Mac OS で環境構築をしてみた.
```
brew install 0xed
brew install dosfstools qemu llvm nasm
echo 'export PATH=/usr/local/Cellar/llvm/12.0.1/bin:$PATH' >> .zshrc
echo 'export PATH=/usr/local/Cellar/dosfstools/4.2/sbin:$PATH' >> .zshrc
```

## EDK Ⅱ の準備
```
cd $HOME
git clone https://github.com/tianocore/edk2.git
cd edk2
git checkout 38c8be123aced4cc8ad5c7e0da9121a181b94251
git submodule init
git submodule update
cd BaseTools/Source/C
make
```

## 環境のチェック
このサンプルのプログラムの場所に移動してから,以下のコマンドを実行。
```
cd /Dropbox/orange_os
curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/day01/bin/hello.efi
qemu-img create -f raw karaage.img 200M
mkfs.fat -n 'KARAAGE OS' -s 2 -f 2 -R 32 -F 32 karaage.img
hdiutil attach -mountpoint mnt karaage.img
mkdir -p mnt/EFI/BOOT
cp hello.efi mnt/EFI/BOOT/BOOTX64.EFI
hdiutil detach mnt
curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/devenv/OVMF_CODE.fd
curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/devenv/OVMF_VARS.fd
```
バイナリファイルを書き込んだイメージをQEMUで実行
```
qemu-system-x86_64 -drive if=pflash,file=OVMF_CODE.fd -drive if=pflash,file=OVMF_VARS.fd -hda karaage.img
```

## C言語での Hello World
```
curl -O https://raw.githubusercontent.com/uchan-nos/mikanos-build/master/day01/c/hello.c
clang -target x86_64-pc-win32-coff -mno-red-zone -fno-stack-protector -fshort-wchar -Wall -c hello.c
lld-link /subsystem:efi_application /entry:EfiMain /out:hello.efi hello.o
```

##　 参考資料
以下のサイトと同じようにしたほうがうまく行った。
https://qiita.com/yamoridon/items/4905765cc6e4f320c9b5