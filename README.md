# vyos-arm64-autobuild

build vyos arm64 rolling by github actions

## 参考

-   https://docs.vyos.io/en/latest/contributing/build-vyos.html

-   https://forum.vyos.io/t/build-iso-error/10640/8

-   https://docs.influxdata.com/telegraf/v1/install/

## 环境要求

-   一台 arm64 的电脑，装有 docker

-   互联网连接

## 构建过程

```bash
docker pull vyos/vyos-build:current-arm64
git clone -b current --single-branch https://github.com/vyos/vyos-build
cd vyos-build
# GPG Key
curl -s https://repos.influxdata.com/influxdata-archive_compat.key > influxdata-archive_compat.key
# config.boot.default默认添加了eth0的dhcp并打开ssh
cp ../config.boot.default data/live-build-config/includes.chroot/opt/vyatta/etc/config.boot.default
# ./data/architectures/arm64.toml 缺少telegraf安装源，需要增加一行 "deb [arch=arm64] https://repos.influxdata.com/debian stable main"
# cat data/architectures/arm64.toml
# additional_repositories = [
#   "deb [arch=arm64] https://repo.saltproject.io/py3/debian/11/arm64/3005 bullseye main",
#   "deb [arch=arm64] https://repos.influxdata.com/debian stable main"
# ]
# ...

# 在增加"deb [arch=arm64] https://repos.influxdata.com/debian stable main"之后
docker run --rm -it --privileged --sysctl net.ipv6.conf.lo.disable_ipv6=0 -v $(pwd):/vyos -w /vyos vyos/vyos-build:current-arm64 bash
# 在docker中
./build-vyos-image iso --architecture arm64 --build-by "canoziia@projectk.org" --custom-apt-key /vyos/influxdata-archive_compat.key
exit
# 宿主机中
ls -l vyos-build/build/vyos-1.5-rolling-xxxxxxxxxxxx-arm64.iso
```

## 启动

-   pve 中运行请添加串口，使用串口显示，并尝试各种 bios/uefi 设置，如果出现 grub 界面即为启动成功。

-   启动后会卡在 exiting boot services vyos，是正常现象，因为显示卡住了，**不是 iso 有问题**。请将其 eth0 连接到路由器上并找到 vyos 通过 dhcp 获得的 ip 地址，ssh 登录即可。
