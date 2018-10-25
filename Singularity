BootStrap: docker
From: centos:6.6

%post
    # nss is needed to fix an SSL bug with curl
    # see https://stackoverflow.com/a/40918004
    yum install -y wget gcc-c++ git gmp-devel cairo-devel bzip2 bzip2-devel xz-devel pkgconfig nss patch

    wget -qO- https://get.haskellstack.org/ | sh

    git clone -b singularity --depth 1 https://github.com/unode/ngless /opt/ngless_src
    cd /opt/ngless_src
    make
    make install prefix=/opt/ngless
    
    # cleanup for smaller image
    cd
    yum remove -y gmp-devel cairo-devel bzip2-devel xz-devel gcc-c++
    yum install -y gmp cairo bzip2 xz
    rm -rf /opt/ngless_src
    rm -rf /usr/local/bin/stack
    rm -rf ~/.stack

%environment
    export PATH=/opt/ngless/bin:$PATH
    export XDG_RUNTIME_DIR=""

%apprun ngless 
    ngless "$@"

%runscript
    ngless "$@"
