# Docker イメージの作成

Raspberry Pi OS の Docker イメージを作成する。
Docker イメージで利用する OS は、Raspberry Pi OS (bullseye) とする。
ARM アーキテクチャなので、古い Docker では、動作しない可能性がある。

- Docker Desktop 4.31.1
- Docker 26.1.4
- Raspberry Pi OS (bullseye)

# Raspberry Pi OS を Docker イメージとしてインポートする

Raspberry Pi OS (bullseye) の圧縮ファイル（root.tar.xz）を以下のページからダウンロードする。
https://downloads.raspberrypi.org/raspios_lite_armhf/

Docker にインポートする

```
docker image import root.tar.xz raspios_bullseye_lite_arm:2022-04-04
```

コンテナを起動して、動作確認する。
起動できれば問題なし。

```
docker run -it --rm --entrypoint /bin/bash raspios_bullseye_lite_arm:2022-04-04
exit
```

# 各種セットアップ済みのイメージを作成する

前述の Raspberry Pi OS のイメージから、各種セットアップ済みのイメージを作成する。
Dockerfile には、Go 言語や、OpenCV などのライブラリを追加でインストール設定してある。
必要に応じてライブラリを追加などをする。

```
cd docker
docker build -t raspios_bullseye_lite_arm_dev .
cd ..
```

GoCV の動作確認をする。
main.go の実行には結構な時間がかかる。

```
docker run -it --rm raspios_bullseye_lite_arm_dev /bin/bash
cd
mkdir opencv_test
cd opencv_test
go mod init example.com/opencv_test
go get -u -d gocv.io/x/gocv@v0.30.0
cd /root/go/pkg/mod/gocv.io/x/gocv@v0.30.0
go run cmd/version/main.go
# コンパイルに数分かかる

gocv version: 0.30.0
opencv lib version: 4.5.1

exit
```
