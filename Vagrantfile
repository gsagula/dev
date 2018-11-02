# -*- mode: ruby -*-
# vi: set ft=ruby :

# Place this file into envoy folder.
#
# $ vagrant up
# $ vagrant ssh
# $ cd /vagrant
# $ bazel build //source/exe:envoy-static

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  
  config.vm.provider "virtualbox" do |v|
    v.name = "envoy_dev"
    v.gui = false
    v.memory = 9192
    v.cpus = 4
  end

  config.vm.box_check_update = false
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  # config.vm.network "private_network", ip: "192.168.0.10"

  config.vm.synced_folder "./", "/vagrant", SharedFoldersEnableSymlinksCreate: false

  config.vm.provision "shell", :privileged => false,  inline: <<-SHELL      
    #!/bin/bash
    set -e

    # Repair "==> default: stdin: is not a tty" message
    sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile

    BAZEL_VERSION=0.18.0
    GO_VERSION=1.10

    sudo apt-get -qq install wget -y
    sudo wget --quiet -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add - # Fingerprint: 6084 F3CF 814B 57C1 CF12 EFD5 15CF 4D18 AF4F 7421
    sudo sh -c 'echo "deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-7 main" >> /etc/apt/sources.list'
    sudo sh -c 'echo "deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial-7 main" >> /etc/apt/sources.list'
    
    export DEBIAN_FRONTEND=noninteractive      
    sudo apt-get -qq update
    sudo apt-get -qq install clang-7 lldb-7 lld-7 clang-format-7 \
    libtool cmake realpath automake ninja-build curl pkg-config zip g++ \
    zlib1g-dev unzip python python-pip python-dev build-essential -y

    sudo -H pip install --upgrade pip
    sudo -H pip install --upgrade virtualenv
    sudo -H pip install sphinx sphinxcontrib-httpdomain sphinx_rtd_theme

    if [ -d "$HOME/.go" ] || [ -d "$HOME/go" ]; then
      exit 1
    fi

    echo "Downloading Golang"
    wget --quiet -q -O /tmp/go.tar.gz https://storage.googleapis.com/golang/go$GO_VERSION.linux-amd64.tar.gz  
    if [ $? -ne 0 ]; then
        exit 1
    fi
    echo "Installing Go $GO_VERSION" 
    tar -C "$HOME" -xzf /tmp/go.tar.gz
    mv "$HOME/go" "$HOME/.go"
    touch "$HOME/.bashrc"
    {
      echo '# GoLang'
      echo 'export GOROOT=$HOME/.go'
      echo 'export PATH=$PATH:$GOROOT/bin'
      echo 'export GOPATH=/vagrant'
      echo 'export PATH=$PATH:$GOPATH/bin'
    } >> "$HOME/.bashrc"
    export GOROOT=$HOME/.go
    export PATH=$PATH:$GOROOT/bin
    export GOPATH=/vagrant
    export PATH=$PATH:$GOPATH/bin
    rm -f /tmp/go.tar.gz

    echo "Installing buildifier"
    go get github.com/bazelbuild/buildtools/buildifier
    
    echo "Downloading Bazel $BAZEL_VERSION" 
    curl -s -LO https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh
    sudo chmod u+x bazel-$BAZEL_VERSION-installer-linux-x86_64.sh
    ./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh --user
    rm -f bazel-$BAZEL_VERSION-installer-linux-x86_64.sh
  SHELL
end
