# 跨域总结

## 1.跨域思路

跨域解决方案一般分为两种：前端解决，后端解决

### 1.1 前端解决方案

通过前端解决的思想就是，通过设置中间件把跨域的请求转发一下，其实就是反向代理，
比如 http://1.2.3.4:8099 想要访问豆瓣的接口 http://www.douban.com/v?a=1很显然直接ajax请求
会有跨域问题，但是如果请求的是http://1.2.3.4:8099/api/v?a=1 就不存在跨域

反向代理就是截取 /api 之后的请求 转发到http://www.douban.com/ 服务器上

1. vue react等项目 可以使用 [http-proxy-middleware](https://www.jianshu.com/p/a248b146c55a)

2. 普通项目 就是以下介绍的 本地安装nginx 反向代理跨域

### 1.2 后端解决方案

后端解决方案，一般是需要后端参与

1. jsonp 回调函数

2. CORS 需要后端加头部 但并不是所有浏览器都支持

## 2.本地配置nginx解决跨域

### 2.1 mac/vmware/设置共享文件

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


### 2.2 配置nginx

#### 2.2.1 修改nginx默认服务器根目录

修改配置文件位置：vim /etc/nginx/sites-available/default 

```
location / {
    # 配置共享文件的位置
    root /mnt/hgfs/ftp;

    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
}

```

#### 2.2.2 访问http://XXX 查看配置是否生效（403错误）

403错误可能有两种情况，不要误以为真的没有权限

1. 配置的共享文件下是否有indx.html，没有的话会报错403 Forbidden

2. 真的没有权限 需要修改一下配置 chrod 修改权限 

[解决Nginx出现403 forbidden](https://blog.csdn.net/onlysunnyboy/article/details/75270533)

相当良心的解决方案


#### 2.2.3 配置反向代理

我用的豆瓣随便的一个接口来测试的

```
location /api{
    # 重写
    rewrite  ^.+api/?(.*)$ /$1 break;
    # 配置代理
    proxy_pass   https://api.douban.com;
}

```
 
### 2.3 测试跨域

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="jquery.js"></script>
    <title>Title</title>
</head>
<body>
<div>
    <button id="btn1">测试跨域</button>
    <div id="content"></div>
</div>
<script type="text/javascript">
    $(document).ready(function () {
        $("#btn1").click(function () {
            // 所有豆瓣请求都以 /api 开始  注意相对绝对路径
            $.get("/api/v2/book/1220562", function (data, status) {
                alert("数据: " + JSON.stringify(data) + "\n状态: " + status);
                $("#content").html(JSON.stringify(data))
            });
        });
    });
</script>
</body>
</html>

```

### 2.4 浏览器测试跨域，跨域成功


## 参考

1. [安装vmtools之后在/mnt目录下没有hgfs文件夹](https://blog.csdn.net/theVicTory/article/details/72976164)
2. [vmware设置共享文件夹](https://blog.csdn.net/mingtianwendy/article/details/78393583)
3. [虚拟机找不到/mnt/hgfs挂载目录](https://blog.csdn.net/jazzsoldier/article/details/54971926)