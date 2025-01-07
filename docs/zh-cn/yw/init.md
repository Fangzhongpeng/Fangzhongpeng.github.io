## init.sh    
    #!/bin/bash
    BASE_DIR=`pwd`

    add_user()
    {
        echo  "starting add user ..."
        read -p "Username:" username
        read -p "Password:" password
        useradd $username
        echo $password |passwd --stdin $username
        echo "user created !!!"
    }

    install_software()
    {
        echo "starting install software ..."
        yum install epel-release -y
        yum update -y
        yum install git wget vim-enhanced   net-tools.x86_64 net-tools lrzsz  nmap vim htop iftop iotop gcc gcc-c++ net-tools unzip  zip  telnet lsof -y
        echo "software installed !!!"
    }

    install_python()
    {
        echo "starting install python ..."
        curl -L https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer | bash
        echo "export PATH=\"/root/.pyenv/bin:\$PATH\"">>/etc/profile
        echo "eval \"\$(pyenv init -)\"">>/etc/profile
        echo "eval \"\$(pyenv virtualenv-init -)\"">>/etc/profile
        source /etc/profile
        echo "python installed !!!"
    }
    install_python3()
    {
        yum -y install zlib*
        cd /usr/local/src/
        wget https://mirrors.huaweicloud.com/python/3.7.10/Python-3.7.10.tgz  -O Python-3.7.10.tgz
        tar xf Python-3.7.10.tgz
        cd Python-3.7.10
        ./configure --prefix=/usr/local/python3
        make && make install
        ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3
        ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
        python3 -V
        which python3
    }
    install_nginx()
    {
    yum install -y pcre pcre-devel gcc openssl openssl-devel wget postgresql-devel
    cd /usr/local/src/ && wget ztocwstres.oss-cn-hangzhou.aliyuncs.com/linux_soft/openresty-1.19.9.1.zip  -O openresty-1.19.9.1.zip
    unzip openresty-1.19.9.1.zip && cd openresty-1.19.9.1
    ./configure --prefix=/usr/local/openresty             --with-luajit               --with-http_iconv_module             --with-http_postgres_module  --with-pcre --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module
    make && make install
    /usr/local/openresty/nginx/sbin/nginx
    mkdir -p /ztocwst/nginx/{conf.d,html,logs}
    }
    close_firewalld()
    {
        echo "starting stop firewalld ..."
        systemctl stop firewalld
        systemctl disable firewalld
        echo "firewalld stoped !!!"
    }

    set_hostname()
    {
        echo "start set your hostname ..."
        read -p "输入新主机名（格式：机房-环境-项目-应用-编号）: " hostname </dev/tty
        hostnamectl set-hostname $hostname
        echo "your hostname is set to" $hostname
    }

    close_selinux()
    {
        echo "starting close selinux ..."
        setenforce 0
        sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
        echo "selinux closed !!!"
    }

    install_docker()
    {
    systemctl stop docker.socket && systemctl stop docker &&  yum remove docker-ce docker-ce-cli containerd.io
    yum install -y yum-utils device-mapper-persistent-data lvm2
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum install docker-ce-19.03.15-3.el7  docker-ce-cli-19.03.15-3.el7 -y
    systemctl enable docker
    echo "docker installed !!!!!!!!!!!"
    echo "installing docker-compose ......"
    #wget https://github.com/docker/compose/releases/download/1.23.2/docker-compose-Linux-x86_64 -O /usr/local/src/docker-compose-Linux-x86_64
    #mv /usr/local/src/docker-compose-Linux-x86_64  /usr/bin/docker-compose && chmod +x /usr/bin/docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    echo "docker-compose installed !!!"
    systemctl start docker
    docker version
    }

    set_docker_daemon_json()
    {
        cat >  /etc/docker/daemon.json <<EOF
    {
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"],
    }
    EOF
        systemctl daemon-reload
        systemctl restart docker
        echo "docker  change successful"
    }
    pull_jdk_image()
    {
    docker login --username=admin http://10.200.192.20/  --password=ztyc1234
    docker pull 10.200.192.20/cicd/jdk1.8.0:221
    docker tag 10.200.192.20/cicd/jdk1.8.0:221 jdk1.8.0:221
    docker images
    }
    install_node_exporter()
    {
    cd /usr/local/src/
    wget  https://ztocwstres.oss-cn-hangzhou.aliyuncs.com/linux_soft/install_node_exporter.sh  -O install_node_exporter.sh
    sh install_node_exporter.sh
    }

    change_swap()
    {
        echo "starting change swap ..."
        read -p "please input your swapfile size:" size
        dd if=/dev/zero of=/usr/local/swapfile  bs=$size count=1
        mkswap /usr/local/swapfile
        swapon /usr/local/swapfile
        echo "/usr/local/swapfile     swap                    swap    defaults        0 0" >> /etc/fstab
        echo "swap changed !!!"
    }

    #kernel_optimization()
    set_history()
    {
        cat > /etc/profile.d/history.sh << \EOF
    USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`   #截取ip
    if [ -z $USER_IP ]
    then
    USER_IP=`hostname`
    fi
    HISTTIMEFORMAT="%F %T $USER_IP:`whoami` "    #history新格式
    export HISTTIMEFORMAT
    export HISTSIZE=50000
    EOF
    source /etc/profile.d/history.sh
    }

    install_jdk()
    {
    wget  https://ztocwstres.oss-cn-hangzhou.aliyuncs.com/linux_soft/jdk-8u161-linux-x64.tar.gz -O jdk-8u161-linux-x64.tar.gz
    tar zxvf jdk-8u161-linux-x64.tar.gz -C /usr/local/
    ln -s /usr/local/jdk1.8.0_161 /usr/local/jdk
    cat  > /etc/profile.d/jdk.sh << EOF
    JAVA_HOME=/usr/local/jdk
    CLASSPATH=${JAVA_HOME}/lib/
    PATH=$PATH:${JAVA_HOME}/bin
    export PATH JAVA_HOME CLASSPAT
    EOF
    source /etc/profile.d/jdk.sh
    }
    install_filebeat()
    {
    cd /usr/local/src/ && wget https://ztocwstres.oss-cn-hangzhou.aliyuncs.com/linux_soft/filebeat-7.16.2-x86_64.rpm -O filebeat-7.16.2-x86_64.rpm
    yum install filebeat-7.16.2-x86_64.rpm
    cat >> /etc/hosts << EOF
    115.236.93.195 hz01-op-elk-kafka-01
    115.236.93.196 hz01-op-elk-kafka-02
    115.236.93.197 hz01-op-elk-kafka-03
    EOF
    systemctl enable filebeat
    systemctl start filebeat
    }

    distribute_id()
    {
    ssh-keygen -t rsa
    rpm -qa | grep expect >> /dev/null
    if [ $? -eq 0 ];then
    echo "expect already install."
    else
    yum install expect -y
    fi
    read -p "输入远程主机ip:" ip
    read -p "输入远程主机root密码:" password

    expect -c "
    spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@$ip
            expect {
                    \"*yes/no*\" {send \"yes\r\"; exp_continue}
                    \"*password*\" {send \"$password\r\"; exp_continue}
                    \"*Password*\" {send \"$password\r\";}
            }
        "

    }
    install_ohmyzsh()
    {
        echo "starting install oh-my-zsh ..."
        yum install zsh -y
        sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
        echo "oh-my-zsh installed !!!"
    }

    print_systeminfo()
    {
        echo "**********************************"
        echo "Hostname:" `hostname`
        cat /proc/cpuinfo |grep vmx >> /dev/null
        if [ $? == 0 ]
        then
            echo "Supporting virtualization"
        else
            echo "Virtualization is not supported"
        fi
        echo "Cpu model:" `cat /proc/cpuinfo |grep "model name" | awk '{ print $4" "$5""$6" "$7 ; exit }'`
        echo "Memory:" `free -m |grep Mem | awk '{ print $2 }'` "M"
        echo "Swap: " `free -m |grep Swap | awk '{ print $2 }'` "M"
        echo "Kernel version: " `cat /etc/redhat-release`
        echo "**********************************"
    }

    help()
    {
        echo "1) 更改主机名"
        echo "2) 安装运维常用工具集"
        echo "3) 安装jdk"
        echo "4) 优化history命令"
        echo "5) 安装docker、安装docker-compose、配置加速器与云仓insecure-registries"
        echo "6) 拉取中通云仓基础jdk镜像:jdk1.8.0:221"
        echo "7) 安装基础监控组件"
        echo "8) 安装日志收集器"
        echo "9) 安装python3.7.10"
        echo "10) 安装nginx1.19.9.1"
        echo "11) 分发本机root公钥到远程主机"
        echo "12) 关闭firewalld与selinux"
        echo "13) 帮助"
        echo "14) 退出"
    }


    main()
    {
    cat << EOF
    ---------------------------------------------
    ---------------------------------------------
    |**********centos主机初始化*********|
    ---------------------------------------------
    |**************** 菜 单 页 *****************|
    ---------------------------------------------
    `echo -e "  \033[35m 1)更改主机名 \033[0m"`
    `echo -e "  \033[35m 2)安装运维常用工具集\033[0m"`
    `echo -e "  \033[35m 3)安装jdk\033[0m"`
    `echo -e "  \033[35m 4)优化history命令\033[0m"`
    `echo -e "  \033[35m 5)安装docker 1.19.03、安装docker-compose、配置加速器与云仓insecure-registries\033[0m"`
    `echo -e "  \033[35m 6)拉取中通云仓基础jdk镜像:jdk1.8.0:221\033[0m"`
    `echo -e "  \033[35m 7)安装基础监控组件\033[0m"`
    `echo -e "  \033[35m 8)安装日志收集器\033[0m"`
    `echo -e "  \033[35m 9)安装python3.7.10\033[0m"`
    `echo -e "  \033[35m 10)安装nginx1.19.9.1\033[0m"`
    `echo -e "  \033[35m 11)分发本机root公钥到远程主机\033[0m"`
    `echo -e "  \033[35m 12)关闭firewalld与selinux\033[0m"`
    `echo -e "  \033[35m 13)帮助\033[0m"`
    `echo -e "  \033[35m 14)退出\033[0m"`
    ---------------------------------------------

    EOF
    read -p "  输入你的选择:" num </dev/tty
    case $num in
    1)
    set_hostname
    ;;
    2)
    install_software
    ;;
    3)
    install_jdk
    ;;
    4)
    set_history
    ;;
    5)
    install_docker
    set_docker_daemon_json
    ;;
    6)
    pull_jdk_image
    ;;
    7)
    install_node_exporter
    ;;
    8)
    install_filebeat
    ;;
    9)
    install_python3
    ;;
    10)
    install_nginx
    ;;
    11)
    distribute_id
    ;;
    12)
    close_selinux
    close_firewalld
    ;;
    13)
    help
    ;;
    14)
    exit 0
    ;;
    *)
    echo "Input Error ,Please again !!!"
    exit 1
    ;;
    esac
    }

    main
