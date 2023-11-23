# Clusterfuzz
google模糊测试工具本地部署（Ubuntu server）

# 文件目录
>clash-linux-amd64-v1.18 (1).0.gz  代理软件

## 一，配置代理
安装过程中需要访问google Clound 所以需要使用代理,如果有其他方式获取可跳过此步骤,

1：下载 clash-linux-amd64-v1.18 (1).0.gz 文件

2：解压文件 
    
    gunzip clash-linux-amd64-v1.18\ \(1\).0.gz

3：重命名文件 为 clash
        
    mv 'clash-linux-amd64-v1.18 (1).0' clash

4：添加执行权限

    chmod +x clash

5：执行文件

    ./clash

    第一次执行会在 ~/.config/clash 下面创建 config.yaml Country.mmdb 文件
    config.yaml 文件是代理文件 

6：下载代理服务器数据（下载完成后 重新执行clash文件即开启代理）

    curl https:/xxxx > ~/.config/clash/config.yaml

## 二，设置终端临时的代理（保留上一步的终端，重新启动以一个终端）
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=http://127.0.0.1:7890
    export no_proxy=127.0.0.0/8

    curl -i google.com 测试代理是否成功

## 三，安装Clusterfuzz

1：克隆 clusterfuzz 到本地

    git clone https://github.com/google/clusterfuzz.git
    cd cd clusterfuzz/

2：检查仓库版本 切换到最新版本

    git tag -l
    git checkout tags/v2.6.0

3：启动docker ubuntu镜像 启动容器 

    docker container run --network host --name clusterfuzz -it -v $(pwd):/clusterfuzz skybro/ubuntu-cn:18.04
    当前应该在容器内了

    docker 内配置代理
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=http://127.0.0.1:7890
    export no_proxy=127.0.0.0/8

4：安装基础软件

    apt-get update && \
    apt-get upgrade -y && \
    apt-get autoremove -y && \
    apt-get install -y \
        apt-transport-https \
        build-essential \
        curl \
        gdb \
        libbz2-dev \
        libcurl4-openssl-dev \
        libffi-dev \
        libgdbm-dev \
        liblzma-dev \
        libncurses5-dev \
        libnss3-dev \
        libreadline-dev \
        libssl-dev \
        locales \
        lsb-release \
        net-tools \
        socat \
        sudo \
        unzip \
        util-linux \
        wget \
        zip \
        zlib1g-dev \
        patchelf \
        git \
        vim
5：安装google-cloud-sdk

    CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)" && \
        echo "deb https://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
        curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -  && \
        apt-get update && apt-get install -y google-cloud-sdk

6：安装python3.7

    curl -sS https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz | tar -C /tmp -xzv
    (cd /tmp/Python-3.7.7 && ./configure --enable-optimizations && make altinstall && rm -rf /tmp/Python-3.7.7)
    pip3.7 install --upgrade pip && pip3.7 install wheel && pip3.7 install pipenv

7：golang

    apt install -y golang

8：java

    apt install -y openjdk-8-jdk

9：node npm

    curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
    sudo apt install nodejs

10：gcloud依赖

    apt-get install -y \
      google-cloud-sdk-app-engine-go \
      google-cloud-sdk-app-engine-python \
      google-cloud-sdk-app-engine-python-extras \
      google-cloud-sdk \
      google-cloud-sdk-datastore-emulator \
      google-cloud-sdk-pubsub-emulator

11：python依赖

    cd clusterfuzz
    python3.7 -m pipenv --python 3.7
    python3.7 -m pipenv sync --dev

12：其它依赖

    pipenv shell
    npm config set unsafe-perm true
    npm install -g bower polymer-bundler
    bower install --allow-root
    python butler.py bootstrap

## 启动网页

    python butler.py run_server -b

    第一次运行需要加上-b选项(--bootstrap)，以后就不用了，另外推荐加上--skip-install-deps (不加上每次都安装很多东西，感觉是我们之前依赖都已经装过了，我试过加上没事，有问题再去掉这个选项): python butler.py run_server -b --skip-install-deps，看到下面内容就运行ok了