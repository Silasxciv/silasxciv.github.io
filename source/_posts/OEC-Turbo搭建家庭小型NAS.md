---
layout: post
title: OEC-Turbo搭建家庭小型NAS
comments: true
tags:
  - NAS
categories: 
  - 教程
date: 2025-08-02 21:48:46
---
在B站上看到有篇用[OEC-Turbo搭建NAS](https://www.bilibili.com/video/BV1UAuuzyEZe/?spm_id_from=333.788.top_right_bar_window_history.content.click&vd_source=3e3e11c3c2bdba291315b78beb1be740)的教程，参考着搭建家庭个人影音库。
<!--more-->

## 设备介绍
|产品|OEC-turbo|
|---|---|
|CPU|四核ARM架构处理器|
|内存|4GB|
|系统存储|8GB|
|网络接口|千兆以太网接口 *1|
|硬盘接口|SATA3.0接口*1；支持内置2.5寸硬盘|
|USB|USB3.0*1|
|电源|12V/2A 电源|
|产品尺寸|145mm*90mm*47mm|
|闲鱼收购价格|99元|
|额外购置硬盘价格|500G西数蓝盘，45元|

## 搭建准备
- OEC-Turbo
- SATA硬盘
- 数据线
- 短接线
- 瑞芯微设备驱动
- 瑞芯微烧录工具
- Armbian固件
- MobaCterm

## 上注armbian固件
1. 拆掉设备外壳，露出内部主板
2. 安装瑞芯微设备驱动程序
  - DriverInstall.exe
3. 打开RKDevTool.exe
  - LoaderToDDR项选择“MiniLoaderAll.bin”
  - system项选择体现下载好的“Armbian.img”
4. 断开DC电源，把数据线一头连到PC电脑
5. 先用短接线短接如图的两个触点，再将数据线C口连接到OEC-Turbo
![插入图片](/assets/images/250802_1.jpg)
5. 当RKDevTool.exe提示“发现一个MASKROM设备”时，点击执行
6. RKDevTool.exe提示“下载完成”后，断开OEC-Turbo上的数据线
7. 连接OEC-Turbo上的DC电源和网线，设备自动开机
8. 进入路由器后台，记录名为armbian的设备的IP地址
9. 打开MobaXterm,SSH连接记录的IP地址
  - 用户名：root
  - 密码：1234
10. 根据提示设置Armbian OS的新密码
11. 根据提示选择默认系统脚本，选择bash即可
12. 根据提示选择是否创建用户，可不创建
13. 键入以下命令更新Armbian OS
  ```
    apt update
  ```

## 安装CasaOS
1. 键入以下命令一键安装CasaOS，不需要管5G内存不足的提示（补充1）
  ```
    curl -fsSL https://get.casaos.io | sudo bash
  ```
2. 在浏览器中键入armbian设备的IP地址，进入CasaOS后台
3. 点击“开始”，创建用户
4. 在App Store中键入以下命令（补充2），
  ```
    https://play.cuse.eu.org/Cp0204-AppStore-Play.zip
  ```
5. 安装dkTurbo插件，图标变灰即生效，自动更换可用的Docker源

## 配置文件共享
1. OEC-Turbo关机，拔掉DC电源
2. 插上SATA硬盘，再连接DC电源
3. 在CasaOS后台，选中新添加的硬盘，点击创建存储空间，点击格式化并创建
4. 在文件管理器中，进入新硬盘区域，创建共享文件夹（补充3）

## 搭建影视库
1. 在文件管理器中新建存放影视剧的文件夹，最好细分门类
2. 下载Jellyfin插件
3. 在Jellyfin设置中，将Media路径设置为存放影视剧的总目录
4. 进入Jellyfin，新建管理员账户
5. 按门类添加不同的媒体库，点击下一步
6. 进入Jellyfin控制台，点击“目录"插件
7. 添加MetaShark插件库
  ```
    https://ghfast.top/https://github.com/cxfksword/jellyfin-plugin-metashark/releases/download/manifest/manifest_cn.json
  ```
8. 进入MetaShark插件，点击安装
9. 进入控制台，重启Jellyfin后，MetaShark状态变为Active
10. 点击媒体库，勾选所有MetaShark，并去掉其余选项
11. 进入控制台，点击“扫描所有媒体库”
12. 此时媒体库大部分资源都可展示出封面

## 补充
### 换源安装Docker
    一键安装CasaOS的过程中会拉取Docker，因为访问的关系，可能拉取失败，依次执行以下指令
  - 设置秘钥
    ```
      curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
    ```
  - 增加软件源
    ```
      sudo add-apt-repository \
      "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
    ```
  - 安装docker
    ```
      sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
  - 再次安装casaos
    ```
      curl -fsSL https://get.casaos.io | sudo bash
    ```
### 更换应用库地址
    App Store中键入上述应用库时，可能因为访问的关系，添加不上，可先添加以下应用库，再安装dkTurbo插件
    https://play.cuse.eu.org/Cp0204-AppStore-Play-arm64.zip
### 创建访问用户
    windows访问共享文件夹时需要用户名和密码，需要添加或删除SMB用户
  - 在Armbian OS界面键入以下命令添加admin的用户
    ```
      useradd admin
    ```
  - 为admin用户添加密码
    ```
      smbpasswd -a admin
    ```
  - 删除admin用户密码
    ```
      smbpasswd -x admin
    ```
  - 删除admin用户
    ```
      userdel admin
    ```
  - 将/etc/samba/smb.casa.conf文件中的guest ok = Yes改为No禁用访客账户

### 将Docker数据搬运至SATA硬盘
  - 在SATA硬盘中新建Docker文件夹
  - 依次执行如下命令，注意文件路径
    ```
      systemctl stop docker.socket
      systemctl stop docker
      mv /var/lib/docker /mnt/Storage1_sda1/docker
      ln -s /mnt/Storage1_sda1/docker /var/lib/docker
      systemctl start docker.socket
      systemctl start docker
    ```
### Tailscale内网穿透
    在tailscale主页设置中打开key页面
    获取一个短期90天的authkey
    在casaOS上的tailscale设置中，添加TS_AUTHKEY