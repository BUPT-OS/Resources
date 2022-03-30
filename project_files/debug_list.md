#### vscode使用ssh连接远程服务器/容器

1.在vscode中下载remote-ssh，remote-container插件

2.remote-ssh插件配置config file

![image-20220330121938718](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220330121938718.png)

```bash
Host 519
  HostName 10.112.94.239
  User rtos
  Port 20212
```

如上图所示，Host为自己设置的服务器名称，HostName为服务器ip地址，User为用户名，Port为端口号

3.在本机上使用ssh-keygen生成本机私钥及公钥（存在于C盘用户文件夹下的.ssh文件夹）

4.修改服务器中~/.ssh/authoirzed_keys文件，加入本机的公钥

![image-20220330123434747](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220330123434747.png)

5.在vscode左侧会出现远程资源管理器，直接connect即可

![image-20220330123734547](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220330123734547.png)

6.连接容器需要将上图SSH Targets选为Containers（上两步可能会输入多次密码）

#### vscode使用ssh连接github，编写容器中的代码

1.在容器中配置~/.gitconfig文件，输入自己的邮箱及名称

![image-20220330124517742](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220330124517742.png)

2.在容器中使用ssh-keygen生成私钥及公钥

3.打开个人github主页，在Settings/SSH and GPG keys中New SSH key，添加上一步生成的公钥。

4.在vscode中使用git remote添加git远程仓库

