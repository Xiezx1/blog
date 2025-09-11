相对于archlinux下使用distrobox的优点：
目录区分，使用distrobox，虚拟ubuntu和原系统共用一个home，部分配置可能异常
脚本化，使用此方式搭建编译环境后，可利用arch-chroot传递参数，实现编译脚本化
步骤：
1、使用debootstrap 创建ubuntu的根目录：
sudo debootstrap focal ./ubuntu https://mirrors.tuna.tsinghua.edu.cn/ubuntu     
./ubuntu为先建好的目录
这里用的是清华源，创建的是20.04，变更版本修改focal 如下图，需要变更为24.04，则修改focal为noble
[图片]
2、创建好后，使用sudo mount --bind 挂载ubuntu根目录
如，这里根目录是ubuntu，则运行：
cd ubuntu
sudo mount --bind . .
3、进入ubuntu：
cd ubuntu
sudo arch-chroot .
4、进入ubuntu后，默认是root用户，需要自行创建用户
sudo user add -m -G sudo xiezx11
sudo passwd xiezx11
5、目前使用，当挂载其他文件到ubuntu中，会导致系统异常（未解决）


