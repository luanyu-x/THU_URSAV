# 配置ArduPilot SITL

（环境：ubuntu20.04，python==3.11）
## 安装DroneKit
1. 官网安装教程：https://dronekit-python.readthedocs.io/en/latest/guide/quick_start.html 。
2. 官网提供的“Hello Drone”程序中，`print`均需加上括号（python2的print可以不加括号，python3则需要加括号）。
3. 一些命令例如`python hello.py`可能需要加上`sudo`才能运行。

## 安装ArduPilot

1. 从源码安装，参考CSDN：https://blog.csdn.net/weixin_43234290/article/details/118389641 。
2. 运行完`git submodule update --init --recursive`后，进入参考链接 https://www.ncnynl.com/archives/202106/4257.html ，运行下列命令即可（交叉编译工具链在运行install-prereqs-ubuntu.sh时会装好）：
    
    ```
    Tools/environment_install/install-prereqs-ubuntu.sh -y
    . ~/.profile
    export PATH=$PATH:$HOME/ardupilot/Tools/autotest
    export PATH=/usr/lib/ccache:$PATH
    ```
    

## 安装MAVProxy和pymavlink

1. 同上，参考 https://blog.csdn.net/weixin_43234290/article/details/118389641 。
2. 启动SITL仿真，参考官网：https://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html （注意第一次运行时要加上`w`参数）。
3. MAVProxy打开console和map失败：Failed to load module: No module named ‘console’以及Failed to load module: No module named ‘map’。参考CSDN：https://blog.csdn.net/Dojinz/article/details/134872301 。具体原因是python的几个包没装好，需要删除重装（注意使用pip3不是pip2，网站上提供的代码有误）。
