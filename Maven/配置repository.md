1.Maven定义的中央仓库

${M2_HOME}/lib/maven-model-builder-X.X.X.jar

```xml
<repositories>
  <repository>
    <id>central</id>
    <name>Central Repository</name>
    <url>https://repo.maven.apache.org/maven2</url>
    <layout>default</layout>
    <!-- 不会下载SNAPSHOT构件 -->
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
  </repository>
</repositories>
```

2.pom.xml中配置仓库

```xml
<repositories>
  <repository>
    <id>aliyun-repo</id>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <releases>
      <enabled>true</enabled>
    </releases>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
  </repository>
  <!-- 可以配置多个 -->
  ...
</repositories>
```

3.settings.xml中配置仓库

```xml
<profiles>  
  <profile>  
    <id>aliyun</id>  
    <repositories>
      <repository>
        <id>aliyun-repo</id>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <releases>
          <enabled>true</enabled>
        </releases>
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
      </repository>
    </repositories>
  </profile>
</profiles>
<activeProfiles>
  <activeProfile>aliyun</activeProfile>  
</activeProfiles>
```

---

镜像

镜像本质代理

```xml
<mirrors>
  <mirror>
    <id>aliyun-repo</id>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <!-- 代理中央仓库 -->
    <mirrorOf>central</mirrorOf>
  </mirror>
</mirrors>
```

