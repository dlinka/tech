release插件可以帮助你发布项目,主要包括修改版本,升级版本,deploy项目等功能

POM

    <scm>
        <developerConnection>scm:git:git@github.com:dlinka/XXX.git</developerConnection>
      <tag>HEAD</tag>
    </scm>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>3.0.0-M1</version>
            </plugin>
        </plugins>
    </build>

    <distributionManagement>
        <repository>
            <id>nexus-releases</id>
            <name>Nexus Release Repository</name>
            <url>http://120.77.87.89:3344/nexus/content/repositories/releases/</url>
        </repository>
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <name>Nexus Snapshot Repository</name>
            <url>http://120.77.87.89:3344/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
    
命令

    mvn release:prepare
    mvn release:perform
    
---
