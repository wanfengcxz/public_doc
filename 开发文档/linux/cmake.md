# 安装cmake

linux下直接使用包管理工具安装

```bash
apt update
sudo apt install cmake
```

如果默认安装的版本太低，可以手动去官网下载

![image-20240312215208467](cmake.assets/image-20240312215208467.png)

下载.sh文件和压缩包都行

.sh运行或者.tar解压之后，得到cmake-3.27.9-linux-x86_64

```bash
cd cmake-3.27.9-linux-x86_64
cd bin
./cmake --version

apt-get remove cmake	# 卸载之前安装的cmake
# 软连接 注意是绝对路径 否则会出错
ln -s /home/cmake-3.27.9-linux-x86_64/bin/cmake /usr/bin/cmake
cd /home
cmake --version	# 验证是否成功
```

