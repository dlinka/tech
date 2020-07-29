推送镜像到Docker Hub

    //登录,会提示输入密码
    docker login -u username
    
    //重新打个镜像,增加username
    docker tag image:tag username/image:tag
    
    docker push username/image:tag

---

使用阿里云镜像仓库

1.选择容器镜像服务
![image](https://user-images.githubusercontent.com/4274041/87323106-e784e500-c560-11ea-9dd8-a0058a18b4e2.png)

2.创建一个命名空间
![image](https://user-images.githubusercontent.com/4274041/87323172-008d9600-c561-11ea-8a9a-e4451a732fb5.png)

3.创建仓库
![image](https://user-images.githubusercontent.com/4274041/87323275-2450dc00-c561-11ea-917e-053d22409316.png)
![image](https://user-images.githubusercontent.com/4274041/87323348-3c286000-c561-11ea-8941-b0d0be5c7994.png)

使用命令推送

    //登录
    docker login --username=XXX registry.cn-hangzhou.aliyuncs.com
    
    //做个新的镜像
    docker tag imageId registry.cn-hangzhou.aliyuncs.com/命名空间/镜像仓库:tag
    
    //推送
    docker push registry.cn-hangzhou.aliyuncs.com/命名空间/镜像仓库:tag

---
