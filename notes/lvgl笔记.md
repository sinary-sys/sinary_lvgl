# lvgl笔记

[TOC]



## 1 lvgl教程

### 1.1  [LVGL 简介](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-intro.html)

LVGL(Light and Versatile Graphics Library），轻巧而多功能的图形库)是一个免费的开放源代码图形库，它提供创建具有易于使用的图形元素，精美的视觉效果和低内存占用的嵌入式GUI所需的一切。

#### LVGL主要特性

1. 功能强大的构建块，例如按钮，图表，列表，滑块，图像等。
2. 带有动画，抗锯齿，不透明，平滑滚动的高级图形
3. 各种输入设备，例如触摸板，鼠标，键盘，编码器等
4. 支持UTF-8编码的多语言
5. 多显示器支持，如TFT，单色显示器
6. 完全可定制的图形元素
7. 独立于任何微控制器或显示器使用的硬件
8. 可扩展以使用很少的内存(64 kB闪存，16 kB RAM)进行操作
9. 操作系统，支持外部存储器和GPU，但不是必需的
10. 单帧缓冲区操作，即使具有高级图形效果
11. 用C语言编写，以实现最大的兼容性(与C ++兼容)
12. 模拟器可在没有嵌入式硬件的PC上进行嵌入式GUI设计
13. 可移植到MicroPython
14. 可快速上手的教程、示例、主题
15. 丰富的文档教程
16. 在MIT许可下免费和开源

#### LVGL硬件要求

基本上，每个现代控制器(肯定必须要能够驱动显示器)都适合运行LVGL。LVGL的最低运行要求很低：

- 16、32或64位微控制器或处理器
- 最低 16 MHz 时钟频率
- Flash/ROM:：对于非常重要的组件要求 >64 kB(建议 > 180 kB)
- RAM
  - 静态 RAM 使用量：~2 kB，取决于所使用的功能和对象类型
  - 堆栈： > 2kB(建议 > 8 kB)
  - 动态数据(堆)：> 2 KB(如果使用多个对象，则建议 > 16 kB)。由 lv_conf.h 中的 LV_MEM_SIZE 宏进行设置。
  - 显示缓冲区：> “水平分辨率”像素(建议 > 10× “水平分辨率” )
  - MCU 或外部显示控制器中的一帧缓冲区
- C99或更高版本的编译器
- 具备基本的C(或C ++)知识：指针，结构，回调...

```shell
请注意，内存使用情况可能会因具体的体系结构、编译器和构建选项而异。
```

#### ==LVGL源码布局==

- **./lvgl** 库本身
- **./lv_drivers** 显示和输入设备驱动程序
- **./lv_examples** 示例和演示
- lvgl官方文档网站(https://docs.lvgl.io)
- lvgl官方博客博客站点(https://blog.lvgl.io)
- sim在线模拟器网站(https://sim.lvgl.io)
- lv_sim _... 适用于各种 IDE 和平台的模拟器项目
- lv_port _... 移植到其他开发板
- lv_binding _... 绑定到其他语言
- lv _...移植到其他平台

```shell
其中，lvgl，lv_examples和lv_drivers是最受维护、关注的核心存储库。
```

![LVGL 简介](picture/lvgl笔记/20210330131157-1.png)

#### LVGL更新发行规则

- lvgl核心存储库遵循语义版本控制规则：
- 不兼容的API的主要版本更改。例如。 v5.0.0，v6.0.0
- 次要版本，用于新的但向后兼容的功能。例如。 v6.1.0，v6.2.0
- 修补程序版本，用于向后兼容的错误修复。例如。 v6.1.1，v6.1.2

### 1.2 LVGL 系统框架

应用程序创建 GUI 并处理特定任务的应用程序。

LVGL 本身是一个图形库。我们的应用程序通过调用 LVGL 库来创建 GUI 。它包含一个 HAL （硬件抽象层）接口，用于注册显示和输入设备驱动程序。

驱动程序除特定的驱动程序外，它还有其他的功能，可驱动显示器到 GPU (可选)、读取触摸板或按钮的输入。

根据 MCU ，有两种典型的硬件设置。 一个带有内置 LCD/TFT 驱动器的外围设备，而另一种是没有内置 LCD/TFT 驱动器的外围设备。 在这两种情况下，都需要一个 **帧缓冲区** 来存储屏幕的当前图像。

1. 集成了 TFT/LCD 驱动器的 MCU 如果 MCU 集成了 TFT/LCD 驱动器外围设备，则可以直接通过RGB接口连接显示器。 在这种情况下，帧缓冲区可以位于内部 RAM（如果MCU有足够的RAM）中，也可以位于外部RAM（如果MCU具有存储器接口）中。
2. 如果 MCU 没有集成 TFT/LCD 驱动程序接口，则必须使用外部显示控制器(例如 SSD1963、SSD1306、ILI9341 )。 在这种情况下，MCU 可以通过并行端口，SPI 或通过 I2C 与显示控制器进行通信。 帧缓冲区通常位于显示控制器中，从而为 MCU 节省了大量 RAM 。

### 1.3 [LVGL 建立一个lvgl项目](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-set-up-an-lvgl-project.html)

要在我们的项目中使用 lvgl ，我们起码需要获取到官方的这两个库：

- lvgl(lvgl)核心图形库的官方 GitHub 仓库地址：https://github.com/lvgl/lvgl。
- lvgl(lv_drivers)输入输出设备驱动官方 GitHub 仓库地址：https://github.com/lvgl/lv_drivers

我们可以克隆或下载这两个库的最新版本，将它们复制到我们的项目中，然后进行适配。

- 目录 lvgl 就是 lvgl 的官方图形库
- 目录 lv_drivers 是 lvgl 输入输出设备驱动官方示例配置
- 目录 [lv_examples](https://github.com/lvgl/lv_examples) 是 lvgl 的官方demo(可选，但不要直接使用到实际项目中)

![创建LVGL项目](picture/lvgl笔记/20210331125657-1.png)

#### LVGL配置文件

上面的三个库中有一个类似名为 **lv_conf_template.h** 的配置头文件(template就是模板的意思)。通过它可以设置库的基本行为，裁剪不需要模块和功能，在编译时调整内存缓冲区的大小等等。

1. 将 **lvgl/lv_conf_template.h** 复制到 lvgl 同级目录下，并将其重命名为 `lv_drv_conf.h` 。打开文件并将开头的 `#if 0` 更改为 `#if 1` 以使能其内容。
2. 将 **lv_drivers/lv_drv_conf_template.h** 复制到 lv_drivers 同级目录下，并将其重命名为 `lv_conf.h` 。打开文件并将开头的 `#if 0` 更改为 `#if 1` 以使能其内容。
3. (可选)将 **lv_examples/lv_ex_conf_template.h** 复制到 lv_examples 同级目录下，并将其重命名为 `lv_ex_conf.h` 。打开文件并将开头的 `#if 0` 更改为 `#if 1` 以使能其内容。

![LVGL配置文件](picture/lvgl笔记/20210331125657-2.png)