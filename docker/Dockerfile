FROM raspios_bullseye_lite_arm:2022-04-04

RUN <<EOF
  # update packages
  set -eux
  # replace apt sources
  # see: https://www.raspbian.org/RaspbianMirrors
  sed -i 's@^deb http://raspbian.raspberrypi.org/raspbian/@deb http://ftp.tsukuba.wide.ad.jp/Linux/raspbian/raspbian/@' /etc/apt/sources.list
  apt-get update
  apt-get full-upgrade -y
  # clean up apt
  apt-get autoremove -y
  apt-get clean
  rm -rf /var/lib/apt/lists/*
EOF

RUN <<EOF
  # install go
  set -eux
  # download go archive
  curl -SLo go.linux-armv6l.tar.gz https://go.dev/dl/go1.22.4.linux-armv6l.tar.gz
  tar -C /usr/local -xzf go.linux-armv6l.tar.gz
  rm go.linux-armv6l.tar.gz
EOF

RUN <<EOF
  # install base packages
  set -eux
  # update apt
  apt-get update
  # install packages
  apt-get install -y \
    vim \
    git \
    python3 \
    python3-pip \
    meson \
    m4 \
    cmake \
    ninja \
    ccache \
    nasm \
    yasm

  # clean up apt
  apt-get autoremove
  apt-get clean
  rm -rf /var/lib/apt/lists/*
EOF

RUN <<EOF
  # install additional packages
  set -eux
  # update apt
  apt-get update
  # install packages
  apt-get install -y \
    libopencv-dev \
    libcharls-dev \
    python3-opencv \
    libssl-dev \
    libcurl4-openssl-dev \
    liblog4cplus-dev \
    libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev \
    libgstreamer-plugins-bad1.0-dev \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-tools

  # clean up apt
  apt-get autoremove
  apt-get clean
  rm -rf /var/lib/apt/lists/*
EOF

ENV PATH=$PATH:/usr/local/go/bin

COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
