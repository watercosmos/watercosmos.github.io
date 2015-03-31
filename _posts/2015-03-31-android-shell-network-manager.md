---
title: Android Shell 网络控制相关命令
---



因为实验室项目需要, 最近在 Android 设备上开发一个简易的通信软件, 涉及到设备网络控制, 为了避免学习 java 和 Android 应用开发时间成本过高, 想到了用 shell 命令的方式曲线救国. 下面是搜集整理的一些实用命令, 在一个 Android 4.3 开发设备上和自己的 Android 5.0 Nexus 4 上测试通过, 需要 root 权限.


1\. wpa_supplicant / wpa_cli / hostapd 概述
---------------------------------

[wpa_supplicant][1] 是运行于客户端的 WPA 加密认证工具, 支持 WPA, WPA2 等加密方式(iwconfig 只能支持 WEP 加密方式). 与 wpa_supplicant 相对应, [hostapd][2] 是用于 AP 的加密认证服务器工具. wpa_supplicant 与 hostapd 均作为守护进程运行于后台, 借助前端控制台工具 wpa_cli 和 hostapd_cli 来进行操作, wpa_cli 和 hostapd_cli 分别包含在 wpa_supplicant 与 hostapd 中.

[1]: http://w1.fi/wpa_supplicant/
[2]: http://w1.fi/hostapd/


2\. WiFi / 移动数据开关
---------------------

WiFi 开关:

    svc wifi enable
    svc wifi disable

Cellular Data 开关:

    svc data enable
    svc data disable

当前 WiFi 状态:

    dumpsys wifi | grep -m 1 Wi-Fi


3\. 获取网络信息
----------------

获取 interface 信息:

    netcfg

interface Up/Down:

    netcfg <interface> {up/down}

获得当前 IP:

    ifconfig <interface>
    netcfg | grep <interface>

扫描可用网络:

    wpa_cli scan
    wpa_cli scan_results

获得当前网络 RSSI:

    wpa_cli signal_poll

收发包统计:

    wpa_cli pktcnt_poll

部分设备并不包含 wpa_cli 工具, 可自行安装.

检测当前网络是否可访问互联网:

    var=`ping -c 5 www.baidu.com | grep loss`
    if [[ ${var:23:1} = "0" ]]; then
        echo Internet is available
    fi

笨办法, 通过 ping 百度, 检查丢包数是否为 0 来判断能否连接互联网.


4\. 开启 WiFi AP 移动热点模式
-----------------------------

在 Android 上启动热点比较麻烦 (Linux PC 上也不方便), 但实验室搭档病愈回归后很快就解决了这一难题, 赞.

1.  结束 wpa_supplicant 进程
2.  加载 wlan 模块 (部分设备需要该步骤, Nexus 4 不需要)
3.  开启 hostapd
4.  分配 IP 地址

在我使用的 Android 开发机上, wpa_supplicant 无论如何终止都将自动重启, 无奈只好通过暂时将 /system/bin/wpa_supplicant 重命名为 wpa_supplicant.bak 解决, 关闭 AP 模式后再恢复. 注意 /system 分区默认是只读的, 重命名 wpa_supplicant 前需要将其重新挂载为读写模式.

AP 配置信息事先写入 /data/misc/wifi/hostapd.conf 文件, 可以在系统设置界面完成.

下面是执行脚本:

    # make sure wpa_supplicant is terminated
    # on some devices, execute this:
    mount -o remount,rw /system
    mv /system/bin/wpa_supplicant /system/bin/wpa_supplicant.bak

    var1=`ps | grep wpa_supplicant`
    if [[ $var1 != "" ]]; then
        wpa_sup=${var1:9:7}
        kill $wpa_sup
    fi

    # on some devices like Nexus 4, this is enough:
    svc wifi disable

    # insmod wlan module, this is not necessary on some devices
    insmod /system/lib/modules/wlan.ko
    sleep 1
    netcfg wlan0 up

    # make sure hostapd is not running
    var2=`ps | grep hostapd`
    if [[ $var2 != "" ]]; then
        hostapd=${var2:9:7}
        kill $hostapd
    fi

    #run hostapd daemon in the background
    chmod 770 /data/misc/wifi/hostapd.conf
    hostapd -B /data/misc/wifi/hostapd.conf

    #assign IP to interface wlan0
    ifconfig wlan0 192.168.1.1 netmask 255.255.255.0

关闭 AP 模式只需要 `kill` 掉 hostapd 进程即可. 恢复 wpa_supplicant 文件后执行 `svc wifi enable` 即可以正常模式打开 WiFi 功能. somehow 在我的开发机上 wpa_supplicant 被重命名后再也无法重启进程, reboot 解决一切 (捂脸).


5\. 连接/切换 WLAN 网络
-----------------------

在未保存任何可用 WLAN 网络的情况下 (/data/misc/wifi/wpa_supplicant.conf 内没有网络记录), 打开 WiFi, 确认 wpa_supplicant 正在运行, 输入 `wpa_cli` 进入 wpa_cli 控制台. (如上文提到的扫描网络, 获得 RSSI 等命令, 可在 shell 内直接输入 `wpa_cli <option> [command]` 执行相关命令, 但注意这种方式下需要用单引号包含双引号内的字符串, 如 '"string"'.)

    $ wpa_cli
    # 检查当前网络, 此时应该为空, 若非空, 下面添加网络时注意避开已存在网络 ID
    # 或调用 remove_network <network id> 删除现有网络
    > list_network
    # 扫描并显示扫描结果, 从中获得可用网络的 SSID
    > scan
    > scan_results
    # 添加一个 network id 为 0 的网络
    > add_network 0
    # 以 CMCC 为例, 将网络 0 的 SSID 设置为 CMCC
    > set_network 0 ssid "CMCC"
    # 设置密码
    > set_network 0 psk "password"
    # 开启网络 0
    > enable_network 0
    # 选择网络 0 进行连接
    > select_network 0

此时若从打印日志中发现 CTRL-EVENT-CONNECTED, 则连接成功.

    # 断开连接
    > disconnect
    # 重新连接
    > reconnect
    # 按以上步骤添加新的网络, 选择新网络并连接
    > enable_network <network id>
    > select_network <network id>
    # 注意, select 新网络后会自动 disable 之前的网络
    # 保存当前配置的网络, 下一次打开 WiFi 后将自动连接已保存网络
    > save_config
    # 退出 wpa_cli 控制台
    > quit

若需要保存多个网络, 注意在 `save_config` 以前 enable 所有网络. 另外可以设置各个网络的优先级, 当下一次打开 WiFi 时, 将自动连接优先级高的网络.

    > set_network <network id> priority <num>

保存配置后, 可以打开 wpa_supplicant 的配置文件, 查看配置内容.

    $ cat /data/misc/wifi/wpa_supplicant.conf
    ctrl_interface=wlan0
    update_config=1
    ...

    network={
            ssid="CMCC"
            psk="aaaaaaaa"
            priority=1
    }
