* 使用`ssh-keygen`命令生成密钥

* 使用`ssh-copy-id mo@xxx`将密钥拷贝到远程主机

* 在.ssh路径添加config文件

  ```shell
  Host name(方便记的名字)
      user root 
      hostname 192.168.1.1 
      prot 22
  ```

