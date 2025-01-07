#### 实现免密登录的两种方式


##### 1.在`远程`主机上生成密钥对

* 在远程主机上生成密钥对
    ssh-keygen -m PEM -t rsa
* 远程主机要将自己的公钥 复制到 ~/.ssh/authorized_keys 里边
* 本地主机指定使用远程主机的秘钥登录   ssh -i 远程主机的秘钥 root@ip

##### 2.使用`本地`主机秘钥登录

* 本地主机生成密钥对
ssh-keygen -m PEM -t rsa
*本地主机的公钥id_rsa.pub 复制到远程主机的~/.ssh/authorized_keys 里边
*然后 本地主机使用本地的默认密钥直接登录远程主机   ssh root@ip

第二方式的具体实现如下：

    生成公钥
    ssh-keygen -t rsa  #使用rsa方式加密  一路回车  <span style="color:#333333;">这个命令生成一个密钥对：id_rsa（私钥文件）和id_rsa.pub（公钥文件）。默认被保存在~/.ssh/目录</span
    公钥cp到远程主机
    ssh-copy-id  -i /root/.ssh/id_rsa.pub root@目标主机IP