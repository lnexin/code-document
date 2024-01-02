https://xinzt.ren/%e4%b8%aa%e4%ba%ba%e4%b8%ad%e5%bf%83%e9%a1%b5%e9%9d%a2/1

main commit 2
main c      3


WebDAV是一些网盘提供的协议，今天说一下如何在Ubuntu或CentOS将WebDAV挂载为本地磁盘。

安装所需程序：

**Ubuntu：**

```
sudo apt-get install davfs2 -y```

CentOS：

```
sudo yum install davfs2 -y```

创建挂载目录：

```
sudo mkdir /mnt/WebDAV```

挂载WebDAV服务到本地目录：

```
sudo mount -t davfs -o noexec https://example.com/webdav/ /mnt/WebDAV/```

之后会要求输入账户和密码登信息。挂载成功后，即可当正常磁盘一样访问WebDAV服务了。速度快慢取决于你自身和服务商的网速。

解除挂载方法：

```
sudo umount /mnt/WebDAV```

使用fstab挂载WebDAV：

```
$ cat << EOF | sudo tee -a /etc/fstab

# personal webdav
https://example.com/webdav/ /mnt/WebDAV  davfs _netdev,noauto,user,uid=nobody,gid=nobody 0 0
EOF```

保存账户密码：

```
cat << EOF | sudo tee -a /etc/davfs2/secrets/mnt/dav account passwordEOF
```



webdav
https://xinzt.ren/archives/327


https://shipengliang.com/software-exp/%E5%A6%82%E4%BD%95%E5%9C%A8ubuntu%E6%88%96centos%E5%B0%86webdav%E6%8C%82%E8%BD%BD%E4%B8%BA%E6%9C%AC%E5%9C%B0%E7%A3%81%E7%9B%98.html

https://blog.aayu.today/linux/ubuntu/20220312-3/

smb
https://cn.linux-console.net/?p=14592


