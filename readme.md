
#### mac vmware 设置共享文件

1. 在虚拟机上找到设置，选择共享文件夹

2. mac vmware想使用共享文件必须安装vmware-tools

    2.1 在虚拟机关闭状态下，选择虚拟机，安装vmware-tools
    2.2 在ubuntu中直接使用 apt 安装 之后会介绍
 
3. 安装vmware-tools之后查看 mnt文件下是否有hgfs文件

    3.1 有hgfs说明已经有共享文件的挂载点
    3.2 如果没有hgfs文件 说明安装的vm-tools可能和ubuntu版本有冲突
    
4. 设置共享目录

```
# 查看当前设置的共享文件
#sudo vmware-hgfsclient

# 上述命令可能由于没有安装包报错，所以先安装一下包
# apt-get install open-vm-tools
# apt-get install open-vm-tools-desktop
# apt-get install open-vm-tools-dkms

# 自己手动创建hgfs并挂载
# mkdir /mnt/hgfs
# vmhgfs-fuse .host:/ /mnt/hgfs

# 使用文件查看是否有权限进入 /mnt/hgfs 如果没有使用一下命令
# sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other -o nonempty

# 每次进入系统都需要自己mount 可以配置/etc/fstab

.host:/    /mnt/hgfs       vmhgfs     defaults  0  0 


```
完成以上步骤，你会发现 /mnt/hgfs/XXX XX 为你设置的共享目录
  
  

#### 参考

1. [安装vmtools之后在/mnt目录下没有hgfs文件夹](https://blog.csdn.net/theVicTory/article/details/72976164)
2. [vmware设置共享文件夹](https://blog.csdn.net/mingtianwendy/article/details/78393583)
3. [虚拟机找不到/mnt/hgfs挂载目录](https://blog.csdn.net/jazzsoldier/article/details/54971926)