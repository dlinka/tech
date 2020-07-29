||目录|文件|
|-|-|-|
|r权限|可以查看目录中的内容<br/>ls命令需要这个权限|可以查看文件内容<br/>cat,tail命令需要这个权限|
|w权限|可以在目录下添加、删除文件或者子目录<br/>rm,mkdir命令需要这个权限|可以修改文件内容<br/>vi命令需要这个权限|
|x权限|可以进入目录<br/>cd命令需要这个权限<br/>如果在目录上没有x权限,也就没有办法cd进入这个目录,也就没有办法执行这个目录下的命令|可以执行文件|

---

    chown user file
    chown user:group file
    chgrp group file
    chmod 777 file
    chmod u=rwx,g=rx,o=r file
    chmod u=rwx,go=rx file
    //设置拥有者r权限,同组人删除r权限
    chmod u+r,g-r file
    //设置所有人w权限
    chmod a+w file
