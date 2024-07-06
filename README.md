# Raspberry Pi Cross Build

このプロジェクトは、Raspberry Pi 向けのクロスコンパイル環境を構築するプロジェクトとなる。
対象は、Raspberry Pi 2 v1.1 (ARMv7) となる。
ただし、オプション指定などの変更で、他のバージョンでも動作すると思われる。
Windows 環境上に構築する前提とする。

Raspberry Pi OS を Docker コンテナ上に構築し、ライブラリなどを利用する。
Raspberry Pi OS の Docker のイメージを作成する理由は、実機で同様のことをすると、時間が掛かるため。
ただし、実機との環境差がないように設定する必要あるので注意が必要。
Docker コンテナ上でも、ビルドした実行ファイルを実行できるが、GPIO などは利用できない。
Raspberry Pi OS は、bullseye を利用する。
現時点での最新は、bookworm だが、カーネルの仕様変更で gpio モジュールの扱いが変更になり、
ライブラリ側の対応が追い付いていないため、古いバージョンを利用している。

# ツール

以下のツールは、Windows 上にインストール済みとする。
MSYS2 は、[Scoop](https://scoop.sh/) を利用してインストールする前提とする。
MinGW64 は、MSYS2 から pacman を利用してインストールする前提とする。

- Windows 11
- IntelliJ
- Docker Desktop 4.31.1
- Docker 26.1.4
- Docker Compose 2.27.1
- Git for Windows 2.45.1
- MSYS2
  - MinGW64
    - cmake
    - make
    - ninja
    - ccache
    - python3
    - python3-pip
    - nasm
    - yasm
    - m4
    - meson

MinGW64 を利用している理由は、MSYS2 でもビルド用のツール類をインストールできるが、
現時点では、pkg-config のバージョンが、2.1.1 なので上手く動作しないため。
MinGW64 の pkg-config は、2.2.0 で、こちらは想定通りに動作する。
バージョン 2.2.0 で、環境変数の優先順位や、デフォルトで検索するディレクトリの変更などがあったらしい。

# Docker イメージの作成

作成方法は、[Docker の設定](docker/README.md) を参照。

# Raspberry Pi 用ツールチェインのインストール

ツールチェインを、以下のサイトからダウンロードする。
SYSPROGS(https://gnutoolchains.com/raspberry/)
バージョンは、Raspberry Pi OS (bullseye)の GCC と同じもの(raspberry-gcc10.2.1-r2.exe)を採用した。

ダウンロードしたインストーラを実行する。
インストール先： C:\SysGCC\raspberry（デフォルト値）
ライセンスに同意できるならば、インストールを続行する。

# Raspberry Pi コンテナからファイルをコピーする

コンテナからファイルをコピーする。
lib ディレクトリのコピーは、実機では /usr/lib のリンクとなっているため。
pkgconfig ディレクトリのコピーは、pkg-config コマンドのために行っている。

```
path %PATH%;C:\SysGCC\raspberry\bin;%USERPROFILE%\scoop\apps\msys2\current\mingw64\bin

del /s /q C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\lib\* > nul
rmdir /s /q C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\lib > nul
del /s /q C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\lib\* > nul
rmdir /s /q C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\lib > nul
del /s /q C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\include\* > nul
rmdir /s /q C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\include > nul

docker run --rm raspios_bullseye_lite_arm_dev tar --use-compress-program pigz --hard-dereference -chf - -C / usr/lib/ usr/include/ | tar -zxf - -C C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot
xcopy /e C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\lib\ C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\lib\ > nul
xcopy /e C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\lib\arm-linux-gnueabihf\pkgconfig\ C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\lib\pkgconfig\ > nul
```

pkg-config コマンドのために、pkgconfig ディレクトリをコピーする。

```
copy C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\lib\arm-linux-gnueabihf\pkgconfig C:\SysGCC\raspberry\arm-linux-gnueabihf\sysroot\usr\lib\pkgconfig
```

# C 言語ソースをクロスコンパイルをする

コンパイルには、CMake を利用する。
ビルドツール類は、MinGW64 を利用し、ツールチェインは、SYSPROGS を利用する。
ポイントは、ツールチェインへのパスを指定すること、pkg-config のパスを指定すること、
および、CMakeLists.txt 内でインポートしている toolchain.cmake ファイル内で、
ツールチェインを設定していることとなる。

```
cd hello-cross-compile
path %PATH%;C:\SysGCC\raspberry\bin;%USERPROFILE%\scoop\apps\msys2\current\mingw64\bin
set PKG_CONFIG_LIBDIR=C:/SysGCC/raspberry/arm-linux-gnueabihf/sysroot/usr/lib/pkgconfig

cmake -G "Ninja" -DCMAKE_VERBOSE_MAKEFILE=ON -B build -S .
cmake --build build -j 12
cd ..
```

ビルドに成功すると、`hello-cross-compile/build/raspberry_pi` 実行ファイルが生成される。
実機の Raspberry Pi または、Docker コンテナ上で動作確認できる。 

# Go 言語ソースをクロスコンパイルする

## CGO を利用しない場合

CGO を利用せず、純粋に Go ソースのみの場合は、MinGW64 や、ツールチェイン関連の設定は不要となる。

```
cd hello-world-go

set GOOS=linux
set GOARCH=arm
set GOARM=7

go build -o hello-world-go
```

生成された、hello-world-go ファイルを Raspberry Pi にコピーして実行する。

リリースビルドの場合は、以下のようにする。

```
go build -o hello-world-go -ldflags="-s -w" -trimpath
```

## CGO を利用する場合

サンプルでは、GoCV を利用する。
コンパイル時に OpenCV の C ライブラリを利用する。
GoCV のバージョンは、Raspberry Pi OS の OpenCV 4.5.1 とマイナーバージョンまでは一致するように、
0.30.0 （OpenCV 4.5.5）としている。

```
cd hello-gocv

set GOOS=linux
set GOARCH=arm
set GOARM=7
set CC=ccache arm-linux-gnueabihf-gcc
set CXX=ccache arm-linux-gnueabihf-g++
set CGO_ENABLED=1
set PKG_CONFIG_LIBDIR=C:/SysGCC/raspberry/arm-linux-gnueabihf/sysroot/usr/lib/pkgconfig
path %PATH%;C:\SysGCC\raspberry\bin;%USERPROFILE%\scoop\apps\msys2\current\mingw64\bin

go build -o hello-gocv

exit
```

ビルドが完了すると、`hello-gocv/hello-gocv` ファイルが生成される。
実機の Raspberry Pi または、Docker コンテナ上で動作確認できる。

ファイルの確認を行う。

```
file hello-gocv
hello-gocv: ELF 32-bit LSB executable, ARM, EABI5 version 1 (GNU/Linux), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, BuildID[sha1]=a05f1b3cb9d499bfde7828addde3ab05169072e8, for GNU/Linux 3.2.0, with debug_info, not stripped
```

リリースビルドの場合は、以下のようにする。

```
go build -o hello-gocv -ldflags="-s -w" -trimpath
```

リリースビルドの場合は、ファイルサイズが 1.4 MB 程度で、オプションを付けない場合は、9 MB 程度となる。

