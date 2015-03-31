---
title: i3 窗口管理器体验
---



一个多月前在 IPN [<内核恐慌>][1] 中第一次听到 Tiling Window Manager 的概念, 意识到这不正是我从未了解但一直存在的需求吗 ?

[1]: http://ipn.li/kernelpanic/

1\. 什么是 Tiling Window Manager
--------------------------------

窗口管理器是一种在图形界面中控制窗口位置与外观的软件, 与桌面环境一起提供, 共同构成了操作系统的用户图形界面. 各类图形界面元素, 如图标, 窗口, 桌面部件, 壁纸等由桌面环境提供, 窗口管理器仅负责控制窗口大小, 位置, 以及边框, 标题栏等元素的外观和行为, 因此窗口管理器通常很少耗费系统资源.

平铺式 (tiling) 窗口管理顾名思义, 像地板砖一样, 将所有窗口以一定比例, 尺寸互不重叠地将屏幕铺满. 通常操作系统提供的桌面管理方式是堆叠式的 (floating), 即各个窗口可以相互叠放在一起, Windows 系统甚至令人发指的在任务栏中都默认合并窗口, 使同类别窗口堆叠在一起 (装系统后第一件事: 取消任务栏合并=.=). 视频播放器, 截图, 录屏软件通常提供一个总在最前端的功能.


2\. i3
------

在不太舒心地体验过 [awesome][2] wm 之后, [i3][3] 给了我惊喜. awesome 默认提供了七八种窗口模式, 需要学习 Lua 脚本才能修改配置, 复杂度指数级提高. i3 默认只有 tiling 模式, 可一键将指定窗口设置为浮动式, man 手册除去配置文件示例只有短短百行, Mod1 + shift + 方向键几乎涵盖全部功能. 应用启动器通过快捷键呼出控制台输入命令的方式实现, 所有命令输入有补全提示. 我习惯将 Mod1 设置为 alt 键, 以我的配置文件为例, i3 的主要使用操作有:

-   alt + d:             呼出控制台, 输入命令启动应用
-   alt + Enter:         打开 terminal
-   alt + <num>...:      切换工作区
-   alt + shift + <num>: 将当前窗口转到指定工作区
-   alt + h/j/k/l:       切换到左/下/上/右窗口
-   alt + h/j/k/l:       将当前窗口移动到屏幕左/下/上/右方
-   alt + (shift) + i:   隐藏 (呼出) 窗口
-   alt + r:             进入 resize 模式, 使用 h/j/k/l 调整窗口大小, Enter 键返回
-   alt + c:             关闭当前窗口
-   alt + Space:         将当前窗口在浮动式/平铺式间切换
-   alt + 鼠标左键拖动:   移动浮动窗口
-   alt + 鼠标右键拖动:   改变浮动窗口大小

以上这些命令基本覆盖日常的百分之九十操作.

[2]: http://awesome.naquadah.org/
[3]: https://i3wm.org/


3\. 配置状态栏
--------------

i3 以一个独立的工具 i3bar 来提供状态栏, 状态栏默认包括左侧的工作区按钮和右侧的状态数据. 注意 tiling 模式没有最小化概念, 当前所有窗口全部平铺在屏幕中, 因此状态栏中自然也不会有类似任务栏或 dock 中的当前窗口按钮.

i3bar 的默认配置文件路径为 ~/.i3status.conf, 以下是我目前的配置, 具体格式可以用 `man i3status` 命令查看手册.

    general {
            colors = true
            interval = 10
    }

    order += "ethernet eth0"
    order += "cpu_usage"
    order += "tztime local"

    ethernet eth0 {
            # if you use %speed, i3status requires root privileges
            format_up = "E: %ip"
            format_down = "E: down"
    }

    tztime local {
            format = "%m-%d %H:%M:%S"
    }

    cpu_usage {
            format = "%usage"
    }


4\. 配置 i3
-----------

i3 的配置脚本非常清晰明了, 默认路径为 ~/.i3/config, 可以用 `man i3` 查看使用说明和配置示例, 通常按个人习惯修改部分快捷键后即可流畅使用. 下面介绍一些初始状态并未开启的实用功能.

    # 开机执行命令, --no-startup-id 参数关闭命令执行通知
    exec [--no-startup-id] command
    # 绑定快捷键以执行指定命令
    bindsym $mod+key exec command

    # 指定名为 "1:      work      " 和 "2:     chrome     " 的工作区
    # 空格用来撑大工作区按钮方便点击
    bindsym $mod+1 workspace 1:      work
    bindsym $mod+2 workspace 2:     chrome
    bindsym $mod+Shift+1 move container to workspace 1:      work
    bindsym $mod+Shift+2 move container to workspace 2:     chrome
    bar {
            ...
            # 使工作区按钮不显示工作区序号, 只显示"      work      "
            strip_workspace_numbers yes
    }

    # 指定类别窗口在启动时自动放入指定工作区
    # 使用 `xprop` 命令获得窗口 class 值
    assign [class="terminal"] → 1:      work
    assign [class="St"] → 1:      work
    assign [class="chrome"] → 2:     chrome
    assign [class="chrome"] → 2:     chrome
    assign [class="shadowsocks"] → 2:     chrome
    assign [class="Nautilus"] → 3:      file
    assign [class="Rhythmbox"] → 3:      file

    # 所有窗口无边框
    for_window [class="^.*"] border none

    # 将当前窗口放入暂存器, 即隐藏
    # 取出暂存器中第一个窗口
    # 通常我会将 shadowsocks 放入 chrome 工作区, 启动后立即隐藏
    bindsym $mod+i move scratchpad
    bindsym $mod+Shift+i scratchpad show


5\. 秀桌面
----------

tiling 桌面无法轻松调整窗口位置和尺寸, 看似掣肘, 实际体验后会发现这种理念大大提高了屏幕利用率和操作便捷性. Linux 通常作为工作机, 预先配置好各个窗口位置, 进入工作状态后用到的所有工具都不需要去翻找调整, 快捷键切换到工作区即可.

通常我会用到上文配置文件提到的三个工作区.

第一屏 work 左侧上下分别为两个 terminal , 分别用来编译和执行查看 man 手册或其他命令, 右侧为 sublime 窗口写代码.

第二屏 chrome 自然就只有 chrome 窗口了, shadowsocks 也放在这里, 但开机启动后便放入暂存区隐藏, 无视掉.

第三屏 file 会打开 nautilus 文件管理器和 rhythmbox 播放器. 再需要往移动存储设备拷贝数据时文件管理器还是要比终端方便很多.在需要阅读 pdf 时会将 pdf 阅读器放在这个工作区, 需要时设为浮动式放到其他区方便对照.

其他临时窗口通通放入第四屏 other 工作区. 这种情况很少发生, 嗯, 工作模式下我还是蛮专心的. (chrome 见证一切 =.=)

![work workspace](/assets/img/2015-03-27-i3-window-manager-work.png)

![chrome workspace](/assets/img/2015-03-27-i3-window-manager-chrome.png)

![file workspace](/assets/img/2015-03-27-i3-window-manager-file.png)


6\. End
-------

一番体验下来, 我对 i3 的满意度可以给 8 分. 当然还有很多功能缺失或我不会用, 比如设定窗口启动位置和大小, 似乎需要另外写脚本去实现, 还没有功夫折腾. 另外 i3 下如何设置屏幕休眠, 如何方便的使用 network manager, 某些软件下如何显示菜单栏, 这些都还没有解决. Anyway, 作为工作机窗口管理器, i3 已经用一种高效简便的方式满足了我的需求.
