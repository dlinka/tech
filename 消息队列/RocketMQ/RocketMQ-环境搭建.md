单机部署RockerMQ

    //官方给的教程是通过source进行安装
    //这里使用Binary安装
    下载地址:https://rocketmq.apache.org/dowloading/releases/
    
    //解压
    unzip rocketmq-all-4.7.1-bin-release.zip
    
    //进入bin
    //修改runserver.sh
    -server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=16m -XX:MaxMetaspaceSize=80m
    //启动name server
    ./mqnamesrv
    
    
    //进入bin
    //修改runbroker.sh
    -server -Xms512m -Xmx512m -Xmn256m
    //启动broker
    ./mqbroker -n localhost:9876
    
安装控制台

    //rocketmq-externals
    git clone git@github.com:apache/rocketmq-externals.git
    
    //修改端口
    //rocketmq-console/src/main/resources/application.properties
    server.port=8027
    
    //打包rocketmq-console
    mvn clean package -DskipTests
    
    //运行
    java -jar rocketmq-console-ng-2.0.0.jar --rocketmq.config.namesrvAddr=127.0.0.1:9876
    
测试

    //修改tools.sh
    export NAMESRV_ADDR=localhost:9876
    
    //发送消息
    ./tools.sh org.apache.rocketmq.example.quickstart.Producer
    
    //消费消息
    ./tools.sh org.apache.rocketmq.example.quickstart.Consumer

---
