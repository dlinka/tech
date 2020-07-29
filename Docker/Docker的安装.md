Centos的安装

    yum install -y yum-utils
    
    //使用阿里云的Docker CE镜像
    //CE是社区版的意思
    yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
    yum install docker-ce docker-ce-cli containerd.io
    
    //运行Docker
    systemctl start docker
    
    //hello world
    docker run hello-world
