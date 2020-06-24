# AWTK针对arm-linux平台的移植。

[AWTK](https://github.com/zlgopen/awtk)是为嵌入式系统开发的GUI引擎库。

[awtk-linux-fb](https://github.com/zlgopen/awtk-linux-fb)是AWTK在arm-linux上的移植。

本项目以[ZLG周立功 linux开发套件 AWork平台iMX287A 入门级ARM9开发板](https://item.taobao.com/item.htm?spm=a230r.1.14.1.29c8b3f8qxjYf7&id=536334628394&ns=1&abbucket=17#detail) 为载体移植，其它开发板可能要做些修改，有问题请请创建issue。 

## 使用方法

* 1.获取源码

> 以下三者并列放在同一个目录。

```
git clone https://github.com/zlgopen/awtk.git
git clone https://github.com/zlgopen/awtk-examples.git
git clone https://github.com/zlgopen/awtk-linux-fb.git
cd awtk-linux-fb
```

* 2.编辑 awtk_config.py 设置工具链的路径

```
TSLIB_LIB_DIR='/opt/28x/tslib/lib'
TSLIB_INC_DIR='/opt/28x/tslib/include'
TOOLS_PREFIX='/opt/28x/gcc-4.4.4-glibc-2.11.1-multilib-1.0/arm-fsl-linux-gnueabi/bin/arm-linux-'
```

* 3.编辑 awtk-port/main\_loop\_linux.c 修改输入设备的文件名

```
#define FB_DEVICE_FILENAME "/dev/fb0"
#define TS_DEVICE_FILENAME "/dev/input/event0"
#define KB_DEVICE_FILENAME "/dev/input/event1"
#define MICE_DEVICE_FILENAME "/dev/input/mouse0"
```

> 注意：在有些平台下，如果设置 #define MICE_DEVICE_FILENAME "/dev/input/mice”，会出现触摸不灵的问题。通过hexdump /dev/input/mice命令发现，按下触摸屏或操作鼠标都会打印信息，即/dev/input/mice会同时接收触摸和鼠标事件。可通过"hexdump  /dev/input/xx" 命令选择正确的鼠标设备文件名。

* 4.编译(请先安装scons)

生成内置 demoui 例子，生成结果在 build/bin 文件夹下的 demoui 文件

```
scons
```

也可以指定生成其他 Demo，生成结果在 build/bin 文件夹下的 demo 文件

```
scons APP=../awtk-examples/HelloWorld-Demo
```

有些Demo包含了两套不同LCD大小的资源，如：Chart-Demo

* 编译Chart-Demo，并使用LCD为800 * 480的资源：

```
scons APP=../awtk-examples/Chart-Demo
```
* 编译Chart-Demo，并使用LCD为480 * 272的资源：

```
scons APP=../awtk-examples/Chart-Demo LCD=480_272
```
* 5.生成发布包

对于内置的 demoui 例子

```
sh ./release.sh
```

对于其他 Demo，需要加入资源文件夹参数，指向应用程序 assets 的父目录

```
sh ./release.sh ../awtk-examples/HelloWorld-Demo/res demo
sh ./release.sh ../awtk-examples/Chart-Demo/res_800_480 demo
```

* 6.运行

把 release.tar.gz 上传到开发板，并解压，然后运行：

```
./release/bin/demoui
./release/bin/demo
```

## 其他问题

#### 1.修改项目路径

默认情况下，scons 脚本假设以下文件夹在同一个目录

```
zlgopen
  |-- awtk
  |-- awtk-examples
  |-- awtk-linux-fb
```

如果实际存放的路径与默认不同，则需要修改以下 awtk-linux-fb/SConstruct 代码，例如：

```
TK_ROOT = joinPath(os.getcwd(), '../awtk')
APP_ROOT=joinPath(os.getcwd(), '../awtk-examples/HelloWorld-Demo')
```

#### 2.使用Direct Rendering Manager (DRM)

缺省使用framebuffer，如果使用DRM，请修改awtk\_config.py，指定drm的路径。

```
#for drm
OS_FLAGS=OS_FLAGS + ' -DWITH_LINUX_DRM=1 -I/usr/include/libdrm '
OS_LIBS = OS_LIBS + ['drm']
```
> DRM目前只在虚拟机中测试过，如果有问题请参考 wtk-port/lcd\_linux\_drm.c 进行调试。

