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

准备lvgl配置文件

![LVGL配置文件](picture/lvgl笔记/20210331125657-3.png)

使能配置文件

lv_conf.h 也可以复制到其他位置，但是应该在编译器选项中添加 `LV_CONF_INCLUDE_SIMPLE` 定义 (例如，对于 gcc 编译器为 `-DLV_CONF_INCLUDE_SIMPLE` ) 并手动设置包含路径。

在配置文件中，注释说明了各个选项的含义。我们在移植时至少要检查以下三个配置选项，其他配置根据具体的需要进行修改：

- `LV_HOR_RES_MAX` 显示器的水平分辨率。
- `LV_VER_RES_MAX` 显示器的垂直分辨率。
- `LV_COLOR_DEPTH` 颜色深度，其可以是：
- 8 - RG332
- 16 - RGB565
- 32 - (RGB888和ARGB8888)

#### 初始化lvgl

准备好这三个库：lvgl、lv_drivers、lv_examples 后，我们就要开始使用lvgl带给我们的功能了。使用 lvgl 图形库之前，我们还必须初始化 lvlg 以及相关其他组件。初始化的顺序为：

1. 调用 lv_init() 初始化 lvgl 库;
2. 初始化驱动程序；
3. 在 LVGL 中注册显示和输入设备驱动程序；
4. 在中断中每隔 `x毫秒` 调用 `lv_tick_inc(x)` 用以告知 lvgl 经过的时间；
5. 每隔 `x毫秒` 定期调用 `lv_task_handler()` 用以处理与 lvgl 相关的任务。

#### Windows初始化示例(Cdoe::Blocks)

如果你是基于 windows上的IDE模拟器(推荐) 进行学习，请先 配置好的项目工程 及 `windows上的IDE模拟器(Cdoe::Blocks)` 用于后面的学习。

```c
#if WIN32
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR szCmdLine, int nCmdShow)
#else
int main(int argc, char** argv)
{
        /*Initialize LittlevGL*/
        lv_init();

        /*Initialize the HAL for LittlevGL*/
        hal_init();

        /*Check the themes too*/
        lv_disp_set_default(lv_windows_disp);

        /*Run your APP here */


#if WIN32
        while(!lv_win_exit_flag) {
#else
        while(1) {
#endif // WIN32
                /* Periodically call the lv_task handler.
                 * It could be done in a timer interrupt or an OS task too.*/
                lv_task_handler();
                usleep(5*1000);       /*Just to let the system breath*/
                lv_tick_inc(5*1000)
        }
        return 0;
}
```

### 1.4 [LVGL 显示接口](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-display-interface.html)

要设置显示，必须初始化 `lv_disp_buf_t` 和 `lv_disp_drv_t` 变量。

- **lv_disp_buf_t** 保存显示缓冲区信息的结构体
- **lv_disp_drv_t** HAL要注册的显示驱动程序、与显示交互并处理与图形相关的结构体、回调函数。

#### LVGL显示缓冲区

`lv_disp_buf_t` 初始化示例：

```c
/*A static or global variable to store the buffers*/
static lv_disp_buf_t disp_buf;

/*Static or global buffer(s). The second buffer is optional*/
static lv_color_t buf_1[MY_DISP_HOR_RES * 10];
static lv_color_t buf_2[MY_DISP_HOR_RES * 10];

/*Initialize `disp_buf` with the buffer(s) */
lv_disp_buf_init(&disp_buf, buf_1, buf_2, MY_DISP_HOR_RES*10);
```

关于缓冲区大小，有 3 种情况：

1. **一个缓冲区** LVGL将屏幕的内保存到缓冲区中并将其发送到显示器。缓冲区可以小于屏幕。在这种情况下，较大的区域将被重画成多个部分。如果只有很小的区域发生变化(例如按下按钮)，则只会刷新该部分的区域。
2. **两个非屏幕大小的缓冲区** 具有两个缓冲区的 LVGL 可以将其中一个作为显示缓冲区，而另一缓冲区的内容发送到后台显示。应该使用 DMA 或其他硬件将数据传输到显示器，以让CPU同时绘图。这样，渲染和刷新并行处理。与 **一个缓冲区** 的情况类似，如果缓冲区小于要刷新的区域，LVGL将按块绘制显示内容
3. **两个屏幕大小的缓冲区** 与两个非屏幕大小的缓冲区相反，LVGL将始终提供整个屏幕的内容，而不仅仅是块。这样，驱动程序可以简单地将帧缓冲区的地址更改为从 LVGL 接收的缓冲区。因此，当MCU具有 LCD/TFT 接口且帧缓冲区只是 RAM 中的一个位置时，这种方法的效果很好。

#### LVGL显示驱动器

一旦缓冲区初始化准备就绪，就需要初始化显示驱动程序。在最简单的情况下，仅需要设置 `lv_disp_drv_t` 的以下两个字段：

- **buffer** 指向已初始化的 `lv_disp_buf_t` 变量的指针。
- **flush_cb** 回调函数，用于将缓冲区的内容复制到显示的特定区域。刷新准备就绪后，需要调用lv_disp_flush_ready()。 LVGL可能会以多个块呈现屏幕，因此多次调用flush_cb。使用 lv_disp_flush_is_last() 可以查看哪块是最后渲染的。

其中，有一些可选的数据字段：

- **hor_res** 显示器的水平分辨率。(默认为 lv_conf.h 中的 `LV_HOR_RES_MAX` )
- **ver_res** 显示器的垂直分辨率。 (默认为 lv_conf.h 中的 `LV_VER_RES_MAX` )
- **color_chroma_key** 在 chrome 键控图像上将被绘制为透明的颜色。(默认为 lv_conf.h 中的 `LV_COLOR_TRANSP` )
- **user_data** 驱动程序的自定义用户数据。可以在 lv_conf.h 中修改其类型。
- **anti-aliasing** 使用抗锯齿(anti-aliasing)(边缘平滑)。缺省情况下默认为 lv_conf.h 中的 `LV_ANTIALIAS` 。
- **rotated** 如果 `1` 交换 `hor_res` 和 `ver_res` 。两种情况下 LVGL 的绘制方向相同(从上到下的线条)，因此还需要重新配置驱动程序以更改显示器的填充方向。
- **screen_transp** 如果为 `1` ，则屏幕可以具有透明或不透明的样式。需要在 lv_conf.h 中启用 `LV_COLOR_SCREEN_TRANSP` 。

要使用GPU，可以使用以下回调：

- **gpu_fill_cb** 用颜色填充内存中的区域。
- **gpu_blend_cb** 使用不透明度混合两个内存缓冲区。
- **gpu_wait_cb** 如果在 GPU 仍在运行 LVGL 的情况下返回了任何 GPU 函数，则在需要确保GPU渲染就绪时将使用此函数。

> 注意，这些功能需要绘制到内存(RAM)中，而不是直接显示在屏幕上。

其他一些可选的回调，使单色、灰度或其他非标准RGB显示一起使用时更轻松、优化：

- **rounder_cb** 四舍五入要重绘的区域的坐标。例如。 2x2像素可以转换为2x8。如果显示控制器只能刷新特定高度或宽度的区域(对于单色显示器，通常为8 px高)，则可以使用它。
- **set_px_cb** 编写显示缓冲区的自定义函数。如果显示器具有特殊的颜色格式，则可用于更紧凑地存储像素。 (例如1位单色，2位灰度等)。这样，lv_disp_buf_t中使用的缓冲区可以较小，以仅保留给定区域大小所需的位数。 set_px_cb不能与两个屏幕大小的缓冲区一起显示缓冲区配置。
- **monitor_cb** 回调函数告诉在多少时间内刷新了多少像素。
- **clean_dcache_cb** 清除与显示相关的所有缓存的回调

要设置 lv_disp_drv_t 变量的字段，需要使用 lv_disp_drv_init(＆disp_drv) 进行初始化。最后，要为 LVGL 注册显示设备，需要调用lv_disp_drv_register(＆disp_drv)。

代码示例：

```c
lv_disp_drv_t disp_drv;                 /*A variable to hold the drivers. Can be local variable*/
lv_disp_drv_init(&disp_drv);            /*Basic initialization*/
disp_drv.buffer = &disp_buf;            /*Set an initialized buffer*/
disp_drv.flush_cb = my_flush_cb;        /*Set a flush callback to draw to the display*/
lv_disp_t * disp;
disp = lv_disp_drv_register(&disp_drv); /*Register the driver and save the created display objects*/
```

回调的一些简单示例：

```c
void my_flush_cb(lv_disp_drv_t * disp_drv, const lv_area_t * area, lv_color_t * color_p)
{
        /*The most simple case (but also the slowest) to put all pixels to the screen one-by-one*/
        int32_t x, y;
        for(y = area->y1; y <= area->y2; y++) {
                for(x = area->x1; x <= area->x2; x++) {
                        put_px(x, y, *color_p)
                        color_p++;
                }
        }

        /* IMPORTANT!!!
         * Inform the graphics library that you are ready with the flushing*/
        lv_disp_flush_ready(disp);
}

void my_gpu_fill_cb(lv_disp_drv_t * disp_drv, lv_color_t * dest_buf, const lv_area_t * dest_area, const lv_area_t * fill_area, lv_color_t color);
{
        /*It's an example code which should be done by your GPU*/
        uint32_t x, y;
        dest_buf += dest_width * fill_area->y1; /*Go to the first line*/

        for(y = fill_area->y1; y < fill_area->y2; y++) {
                for(x = fill_area->x1; x < fill_area->x2; x++) {
                        dest_buf[x] = color;
                }
                dest_buf+=dest_width;    /*Go to the next line*/
        }
}

void my_gpu_blend_cb(lv_disp_drv_t * disp_drv, lv_color_t * dest, const lv_color_t * src, uint32_t length, lv_opa_t opa)
{
        /*It's an example code which should be done by your GPU*/
        uint32_t i;
        for(i = 0; i < length; i++) {
                dest[i] = lv_color_mix(dest[i], src[i], opa);
        }
}

void my_rounder_cb(lv_disp_drv_t * disp_drv, lv_area_t * area)
{
  /* Update the areas as needed. Can be only larger.
   * For example to always have lines 8 px height:*/
   area->y1 = area->y1 & 0x07;
   area->y2 = (area->y2 & 0x07) + 8;
}

void my_set_px_cb(lv_disp_drv_t * disp_drv, uint8_t * buf, lv_coord_t buf_w, lv_coord_t x, lv_coord_t y, lv_color_t color, lv_opa_t opa)
{
        /* Write to the buffer as required for the display.
         * Write only 1-bit for monochrome displays mapped vertically:*/
 buf += buf_w * (y >> 3) + x;
 if(lv_color_brightness(color) > 128) (*buf) |= (1 << (y % 8));
 else (*buf) &= ~(1 << (y % 8));
}

void my_monitor_cb(lv_disp_drv_t * disp_drv, uint32_t time, uint32_t px)
{
  printf("%d px refreshed in %d ms\n", time, ms);
}

void my_clean_dcache_cb(lv_disp_drv_t * disp_drv, uint32)
{
  /* Example for Cortex-M (CMSIS) */
  SCB_CleanInvalidateDCache();
}
```

### 1.5 [LVGL 输入设备接口](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-input-device-interface.html)

#### LVGL输入设备的类型

要设置输入设备，必须初始化 `lv_indev_drv_t` 变量：

```c
lv_indev_drv_t indev_drv;
lv_indev_drv_init(&indev_drv);      /*Basic initialization*/
indev_drv.type =...                 /*See below.*/
indev_drv.read_cb =...              /*See below.*/

/*Register the driver in LVGL and save the created input device object*/
lv_indev_t * my_indev = lv_indev_drv_register(&indev_drv);
```

**类型** (indev_drv.type)可以是：

- **LV_INDEV_TYPE_POINTER** 触摸板或鼠标
- **LV_INDEV_TYPE_KEYPAD** 键盘或小键盘
- **LV_INDEV_TYPE_ENCODER** 带有左，右，推动选项的编码器
- **LV_INDEV_TYPE_BUTTON** 外部按钮按下屏幕

**read_cb** (indev_drv.read_cb)是一个函数指针，将定期调用该函数指针以报告输入设备的当前状态。它还可以缓冲数据并在没有更多数据要读取时返回 `false` ，或者在缓冲区不为空时返回 `true` 。

进一步了解有关 LVGL输入设备 的更多信息。

#### LVGL触摸板，鼠标或任何指针

可以单击屏幕点的输入设备属于此类别。

```c
indev_drv.type = LV_INDEV_TYPE_POINTER;
indev_drv.read_cb = my_input_read;

...

bool my_input_read(lv_indev_drv_t * drv, lv_indev_data_t*data)
{
        data->point.x = touchpad_x;
        data->point.y = touchpad_y;
        data->state = LV_INDEV_STATE_PR or LV_INDEV_STATE_REL;
        return false; /*No buffering now so no more data read*/
}
```

即使状态为 LV_INDEV_STATE_REL ，触摸板驱动程序也必须返回最后的 X/Y 坐标。

要设置鼠标光标，请使用 `lv_indev_set_cursor(my_indev,&img_cursor)` 。( `my_indev` 是 `lv_indev_drv_register` 的返回值)键盘或键盘

#### LVGL触摸板或键盘

带有所有字母的完整键盘或带有一些导航按钮的简单键盘均属于此处。

要使用键盘/触摸板：

- 注册具有 LV_INDEV_TYPE_KEYPAD 类型的 read_cb 函数。
- 在 lv_conf.h 中启用 LV_USE_GROUP
- 必须创建一个对象组：lv_group_t * g = lv_group_create()，并且必须使用 lv_group_add_obj(g，obj) 向其中添加对象
- 必须将创建的组分配给输入设备：lv_indev_set_group(my_indev，g)( my_indev 是 lv_indev_drv_register 的返回值)
- 使用 LV_KEY _... 在组中的对象之间导航。有关可用的密钥，请参见 lv_core/lv_group.h。

```c
indev_drv.type = LV_INDEV_TYPE_KEYPAD;
indev_drv.read_cb = keyboard_read;

...

bool keyboard_read(lv_indev_drv_t * drv, lv_indev_data_t*data){
  data->key = last_key();            /*Get the last pressed or released key*/

  if(key_pressed()) data->state = LV_INDEV_STATE_PR;
  else data->state = LV_INDEV_STATE_REL;

  return false; /*No buffering now so no more data read*/
}
```

#### LVGL编码器

可以通过下面四种方式使用编码器：

1. 按下按钮
2. 长按其按钮
3. 转左
4. 右转

简而言之，编码器输入设备的工作方式如下：

- 通过旋转编码器，可以专注于下一个/上一个对象。
- 在简单对象(如按钮)上按下编码器时，将单击它。
- 如果将编码器按在复杂的对象(如列表，消息框等)上，则该对象将进入编辑模式，从而转动编码器即可在对象内部导航。
- 长按按钮，退出编辑模式。

要使用编码器(类似于键盘)，应将对象添加到组中。

```c
indev_drv.type = LV_INDEV_TYPE_ENCODER;
indev_drv.read_cb = encoder_read;

...

bool encoder_read(lv_indev_drv_t * drv, lv_indev_data_t*data){
  data->enc_diff = enc_get_new_moves();

  if(enc_pressed()) data->state = LV_INDEV_STATE_PR;
  else data->state = LV_INDEV_STATE_REL;

  return false; /*No buffering now so no more data read*/
}
```

#### 使用带有编码器逻辑的按钮

除了标准的编码器行为外，您还可以利用其逻辑来使用按钮导航(聚焦)和编辑小部件。如果只有几个按钮可用，或者除编码器滚轮外还想使用其他按钮，这将特别方便。

需要有3个可用的按钮：

- **LV_KEY_ENTER** 将模拟按下或推动编码器按钮
- **LV_KEY_LEFT** 将向左模拟转向编码器
- **LV_KEY_RIGHT** 将正确模拟转向编码器
- 其他键将传递给焦点小部件

如果按住这些键，它将模拟indev_drv.long_press_rep_time中指定的时间段内的编码器单击。

```c
indev_drv.type = LV_INDEV_TYPE_ENCODER;
indev_drv.read_cb = encoder_with_keys_read;

...

bool encoder_with_keys_read(lv_indev_drv_t * drv, lv_indev_data_t*data){
  data->key = last_key();            /*Get the last pressed or released key*/
                                                                         /* use LV_KEY_ENTER for encoder press */
  if(key_pressed()) data->state = LV_INDEV_STATE_PR;
  else {
          data->state = LV_INDEV_STATE_REL;
          /* Optionally you can also use enc_diff, if you have encoder*/
          data->enc_diff = enc_get_new_moves();
  }

  return false; /*No buffering now so no more data read*/
```

#### LVGL按键

按钮是指屏幕旁边的外部“硬件”按钮，它们被分配给屏幕的特定坐标。如果按下按钮，它将模拟在指定坐标上的按下。 (类似于触摸板)

使用 lv_indev_set_button_points(my_indev, points_array) 将按钮分配给坐标。points_array应该看起来像const lv_point_t points_array [] = {{12,30}，{60,90}，...}

points_array不能超出范围。将其声明为全局变量或函数内部的静态变量。

```c
indev_drv.type = LV_INDEV_TYPE_BUTTON;
indev_drv.read_cb = button_read;

...

bool button_read(lv_indev_drv_t * drv, lv_indev_data_t*data){
        static uint32_t last_btn = 0;   /*Store the last pressed button*/
        int btn_pr = my_btn_read();     /*Get the ID (0,1,2...) of the pressed button*/
        if(btn_pr >= 0) {               /*Is there a button press? (E.g. -1 indicated no button was pressed)*/
           last_btn = btn_pr;           /*Save the ID of the pressed button*/
           data->state = LV_INDEV_STATE_PR;  /*Set the pressed state*/
        } else {
           data->state = LV_INDEV_STATE_REL; /*Set the released state*/
        }

        data->btn = last_btn;            /*Save the last button*/

        return false;                    /*No buffering now so no more data read*/
}
```

#### LVGL其它功能

除了 read_cb 之外，还可以在 lv_indev_drv_t 中指定 feedback_cb 回调。输入设备发送任何类型的事件时，都会调用feedback_cb。 (独立于其类型)。它允许为用户提供反馈，例如在LV_EVENT_CLICK上播放声音。

可以在lv_conf.h中设置以下参数的默认值，但可以在lv_indev_drv_t中覆盖默认值：

- **拖拽限制(drag_limit)** 实际拖动对象之前要滑动的像素数 drag_throw 拖曳速度降低[％]。更高的价值意味着更快的减速
- **(drag_throw)** 拖曳速度降低[％]。更高的价值意味着更快的减速
- **(long_press_time)** 按下时间发送 LV_EVENT_LONG_PRESSED (以毫秒为单位)
- **(long_press_rep_time)** 发送 LV_EVENT_LONG_PRESSED_REPEAT 的时间间隔(以毫秒为单位)
- **(read_task)** 指向读取输入设备的lv_task的指针。可以通过 `lv_task_...()` 函数更改其参数

每个输入设备都与一个显示器关联。默认情况下，新的输入设备将添加到最后创建的或显式选择的显示设备(使用lv_disp_set_default())。相关的显示已存储，并且可以在驱动程序的显示字段中更改。

### 1.6 [LVGL 心跳](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-heartbeat.html)

LVGL心跳，LVGL 需要系统滴答声才能知道动画和其他任务的经过时间。

为此我们需要定期调用 lv_tick_inc(tick_period) 函数，并以毫秒为单位告知调用周期。例如， lv_tick_inc(1) 用于每毫秒调用一次。

为了精确地知道经过的毫秒数，lv_tick_inc 应该在比 lv_task_handler() 更高优先级的例程中被调用(例如在中断中)，即使 lv_task_handler 的执行花费较长时间。

使用 FreeRTOS 时，可以在 vApplicationTickHook 中调用 lv_tick_inc 。

在基于 Linux 的设备上(例如在 Raspberry Pi 上)， lv_tick_inc 可以在如下所示的线程中调用，比如：

```c
void * tick_thread (void *args)
{
          while(1) {
                usleep(5*1000);   /*Sleep for 5 millisecond*/
                lv_tick_inc(5);      /*Tell LVGL that 5 milliseconds were elapsed*/
        }
}
```

### 1.7 [LVGL 任务处理器](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-task-handler.html)

**任务处理器(Task Handler)**要处理 LVGL 的任务，我们需要定期通过以下方式之一调用 lv_task_handler() ：

- mian 函数中设置 while(1) 调用
- 定期定时中断(低优先级然后是 lv_tick_inc()) 中调用
- 定期执行的 OS 任务中调用

计时并不严格，但应保持大约5毫秒以保持系统响应。

范例：

```c
while(1) {
  lv_task_handler();
  my_delay_ms(5);
}
```

### 1.8 [LVGL 睡眠管理](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-management-of-sleep.html)

**LVGL睡眠管理**，没有用户输入时，MCU 可以进入睡眠状态。在这种情况下，mian 函数中的 while(1) 应该看起来像这样：

```c
while(1) {
  /*Normal operation (no sleep) in < 1 sec inactivity*/
  if(lv_disp_get_inactive_time(NULL) < 1000) {
          lv_task_handler();
  }
  /*Sleep after 1 sec inactivity*/
  else {
          timer_stop();   /*Stop the timer where lv_tick_inc() is called*/
          sleep();                  /*Sleep the MCU*/
  }
  my_delay_ms(5);
}
```

如果发生唤醒(按，触摸或单击等)，还应该在输入设备读取功能中添加以下几行：

```c
lv_tick_inc(LV_DISP_DEF_REFR_PERIOD);  /*Force task execution on wake-up*/
timer_start();                         /*Restart the timer where lv_tick_inc() is called*/
lv_task_handler();                     /*Call `lv_task_handler()` manually to process the wake-up event*/
```

除了 lv_disp_get_inactive_time() 外，还可以调用 lv_anim_count_running() 来查看每个动画是否完成。

### 1.9 [LVGL 操作系统和中断](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-operating-system-and-interrupts.html)

LVGL默认情况下 **不是线程安全的** 。

但是，在以下情况中，调用 LVGL 相关函数是有效的：

- 在事件 (Events) 中。在 "事件" 中了解更多信息。
- 在 (lv_tasks) 中。在 "任务" 中了解更多信息。

#### 任务和线程

如果需要使用实际的任务或线程，则需要一个互斥锁，该互斥锁应在调用 lv_task_handler 之前被调用，并在其之后释放。同样，必须在与每个LVGL(lv _...)相关的函数调用和代码周围的其他任务和线程中使用相同的互斥锁。这样，就可以在真正的多任务环境中使用LVGL。只需使用互斥锁(mutex)即可避免同时调用 LVGL 函数。

#### 中断

避免从中断中调用 LVGL 函数( lv_tick_inc() 和 lv_disp_flush_ready() 除外)。但是，如果需要执行此操作，则必须在 lv_task_handler 运行时禁用 LVGL 函数的中断。设置标志或某个值并在 lv_task 中定期检查是一种不错的方法。

### 1.10 [LVGL 日志记录](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-logging.html)

LVGL 内置有日志模块，用于记录用户库中正在发生的事情。

#### LVGL日志级别

要启用日志记录，需要在 `lv_conf.h` 中将 **LV_USE_LOG** 设置为 `1` ，并将 **LV_LOG_LEVEL** 设置为以下值之一：

- **LV_LOG_LEVEL_TRACE** 记录所有信息
- **LV_LOG_LEVEL_INFO** 记录重要事件
- **LV_LOG_LEVEL_WARN** 记录是否发生了警告事件
- **LV_LOG_LEVEL_ERROR** 记录错误信息，当系统可能发生故障时或致命错误
- **LV_LOG_LEVEL_NONE** 不要记录任何东西

级别高于设置的日志级别的事件也将被记录。例如。如果使用 **LV_LOG_LEVEL_WARN** ，也会记录错误。

#### LVGL使用printf记录

如果您的系统支持printf，则只需在 `lv_conf.h` 中启用 **LV_LOG_PRINTF **即可发送带有 printf 的日志。

#### LVGL自定义日志功能

如果不能使用 printf 或想要使用自定义函数进行日志记录，可以使用 lv_log_register_print_cb() 注册 "logger" 回调。

例如：

```c
void my_log_cb(lv_log_level_t level, const char * file, int line, const char * fn_name, const char * dsc)
{
  /*Send the logs via serial port*/
  if(level == LV_LOG_LEVEL_ERROR) serial_send("ERROR: ");
  if(level == LV_LOG_LEVEL_WARN)  serial_send("WARNING: ");
  if(level == LV_LOG_LEVEL_INFO)  serial_send("INFO: ");
  if(level == LV_LOG_LEVEL_TRACE) serial_send("TRACE: ");

  serial_send("File: ");
  serial_send(file);

  char line_str[8];
  sprintf(line_str,"%d", line);
  serial_send("#");
  serial_send(line_str);

  serial_send(": ");
  serial_send(fn_name);
  serial_send(": ");
  serial_send(dsc);
  serial_send("\n");
}

...

lv_log_register_print_cb(my_log_cb);
```

#### LVGL添加日志

还可以通过 **LV_LOG_TRACE/INFO/WARN/ERROR(description)** 函数使用日志模块。

### 1.11 [LVGL 对象](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-objects.html)

在 LVGL 中，用户界面的基本构建块是对象，也称为小部件(widget)。例如，按钮，标签，图像，列表，图表或文本区域。

查看[极客笔记](https://deepinout.com/)中的 LVGL所有的对象类型(widget) 。

#### LVGL对象的属性(Attributes)

#### 对象的基本属性

所有对象类型都共享一些基本属性：

- Position (位置)
- Size (尺寸)
- Parent (父母)
- Drag enable (拖动启用)
- Click enable (单击启用)
- position (位置)
- 等等

我们可以使用 `lv_obj_set _...` 和 `lv_obj_get _...` 等前缀的函数设置或者获取这些属性。例如：

```c
/* 设置基础对象的属性 */
lv_obj_set_size(btn1, 100, 50);   /* 设置按键的大小 */
lv_obj_set_pos(btn1, 20,30);      /* 设置按键的位置 */
```

#### 对象的特殊属性

有些对象类型也具有特殊的属性。例如，滑块具有

- Min. max. values (最小最大值)
- Current value (当前值)
- Custom styles (自定义样式)

对于这些属性，每种对象类型都有唯一的 API 函数。例如一个滑块的 API 调用过程：

```c
/* 设置滑块的特殊属性 */
lv_slider_set_range(slider1, 0, 100);                   /* 设置滑块的最小值和最大值 */
lv_slider_set_value(slider1, 40, LV_ANIM_ON);   /* 设置当前值(屏幕坐标系位置) */
lv_slider_set_action(slider1, my_action);       /* 设置回调函数 */
```

要查看 API 的实现代码，可以检查相应的头文件（例如滑块对象的头文件 `lv_objx/lv_slider.h`）

#### LVGL对象的工作机制

亲子结构

父对象可以作为其子对象的容器。每个对象只能一个父对象（**屏幕除外**），但是一个父对象可以有无限多个子对象。父对象的类型没有限制，但是有特殊的父对象（例如，按钮）和特殊的子对象（例如，标签）。

#### LVGL追随原则

如果更改了父对象的位置，则子对象将与父对象一起移动，并且子对象的位置都保持相对于父对象位置不变。 例如，坐标 (0,0) 表示子对象将独立于父对象的位置保留在父对象的左上角，代码：

![LVGL追随原则](picture/lvgl笔记/20210401124733-1.png)

一个父子对象

```c
lv_obj_t * par = lv_obj_create(lv_scr_act(), NULL); /* 在当前屏幕中创建一个对象 */
lv_obj_set_size(par, 100, 80);                      /* 设置对象的大小 */

lv_obj_t * obj1 = lv_obj_create(par, NULL);         /* 基于前面创建的对象(par)创建一个子对象(obj1)，之前的对像成为父对象 */
lv_obj_set_pos(obj1, 10, 10);                       /* 设置子对象的位置 */
```

当我们修改父对象的位置，子对象也会一起移动，以保持和父对象的相对位置不变：

![LVGL追随原则](picture/lvgl笔记/20210401124733-2.png)

子对象跟随父对象

```c
lv_obj_set_pos(par, 50, 50);    /* 移动父对象，子对象也会跟着移动，以保持相对位置不变 */
```

#### 子对象仅在父对象的范围内可见

如果子对象的部分或全部不在其父级之内，则超出父对象的部分将不可见。

![LVGL追随原则](picture/lvgl笔记/20210401124733-3.png)

子对象超出父对象的部分不可见

```c
lv_obj_set_x(obj1, -30);        /* 将子对象移出一部分到从父对象的范围内之外 */
```

#### 创建-删除对象

在LVGL中，可以在运行时动态地创建和删除对象。这意味着仅当前创建的对象需要消耗RAM。例如，如果需要图表，我们可以在需要时创建它，并在不可见或不需要时将其删除。

每个对象类型都有各自的创建函数。它需要两个参数：

- 指向父对象的指针。创建屏幕时以 NULL 作为父级。
- 用于复制具有相同类型的对象的指针(可选)。如果不行进行复制操作为 NULL。

使用 lv_obj_t 指针作为句柄在 C 代码中引用所有对象。以后可以使用该指针设置或获取对象的属性。

创建函数如下所示：

```c
lv_obj_t * lv_ <type>_create(lv_obj_t * parent, lv_obj_t * copy);
```

所有对象类型都有一个通用的删除功能。它删除对象及其所有子对象。

```c
void lv_obj_del(lv_obj_t * obj);
```

`lv_obj_del` 将立即删除该对象。如果出于某种原因不能立即删除该对象，则可以使用 `lv_obj_del_async(obj)` ，例如，如果要删除子对象的 LV_EVENT_DELETE 信号中对象的父对象，这很有用。

我们可以使用 `lv_obj_clean` 删除对象的所有子对象（但不会删除对象本身）：

```c
void lv_obj_clean(lv_obj_t * obj);
```

#### LVGL屏幕对象

创建屏幕对象

屏幕是没有父对象的特殊对象。应该像这样创建它们：

```c
lv_obj_t * scr1 = lv_obj_create(NULL, NULL);
```

可以使用任何对象类型创建屏幕。例如：创建墙纸的基础对象或图像。

获取活动屏幕

这始终是每个显示屏上的活动屏幕。默认情况下，该库为每个显示创建并加载 “基础对象” 作为屏幕。

要获取当前活动的屏幕使用函数 `lv_scr_act()`

载入屏幕

调用函数 `lv_scr_load(scr1)` 加载屏幕。

加载屏幕动画

我们可以调用函数： `lv_scr_load_anim(scr, transition_type, time, delay, auto_del)` 加载屏幕动画。参数 `transition_type` 是动画过渡类型，该参数可设为：

- `LV_SCR_LOAD_ANIM_NONE` 延迟x毫秒后立即切换
- `LV_SCR_LOAD_ANIM_OVER_LEFT/RIGHT/TOP/BOTTOM` 将新屏幕移到给定方向上
- `LV_SCR_LOAD_ANIM_MOVE_LEFT/RIGHT/TOP/BOTTOM` 将旧屏幕和新屏幕都移至给定方向
- `LV_SCR_LOAD_ANIM_FADE_ON` 使新屏幕淡出旧屏幕

将 `auto_del` 设置为 `true` 会在动画结束时自动删除旧屏幕。

在延迟时间之后开始动画播放时，新屏幕将变为活动状态（由 `lv_scr_act()` 返回）。

处理多个显示

屏幕在当前选择的默认屏幕上创建。默认显示设备使用 `lv_disp_drv_register` 注册的最后一个屏幕作为显示，或者可以使用 `lv_disp_set_default(disp)` 显式选择新的默认显示屏幕。

`lv_scr_act()` , `lv_scr_load()` 和 `lv_scr_load_anim()` 将会在默认的屏幕上操作。

访问 多显示器支持 以了解更多信息。

#### LVGL零件(Parts)

widget 可以包含多个 Parts 。例如，按钮仅具有主要部分，而滑块则由背景，指示器和旋钮组成。

Parts 名称的构造类似于 `LV_ + <TYPE> _PART_ <NAME>` 。比如 `LV_BTN_PART_MAIN` 、 `LV_SLIDER_PART_KNOB` 。 通常在将样式添加到对象时使用 Parts。使用 Parts 可以将不同的样式分配给对象的不同 Parts 。

#### LVGL状态-States

对象可以处于以下状态的组合：

- **LV_STATE_DEFAULT** 默认或正常状态
- **LV_STATE_CHECKED** 选中或点击
- **LV_STATE_FOCUSED** 通过键盘或编码器聚焦或通过触摸板/鼠标单击
- **LV_STATE_EDITED** 由编码器编辑
- **LV_STATE_HOVERED** 鼠标悬停（现在还不支持）
- **LV_STATE_PRESSED** 按下
- **LV_STATE_DISABLED** 禁用或无效

当用户按下，释放，聚焦等对象时，状态通常由库自动检测更改。 当然状态也可以手动检测更改。 要完全覆盖当前状态，调用 `lv_obj_set_state(obj, part, LV_STATE...)` 要设置或清除某个状态(但不更改其他状态)，调用 `lv_obj_add/clear_state(obj, part, LV_STATE_...)` 可以组合使用状态值。例如： `lv_obj_set_state(obj, part, LV_STATE_PRESSED | LV_PRESSED_CHECKED)` .

要了解有关状态的更多信息，请阅读[极客笔记](https://deepinout.com/)中的 LVGL 样式(Styles) 概述的相关部分。

### 1.12 [LVGL 对象层级](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-objects-layers.html)

#### LVGL创建对象层级顺序

默认情况下，LVGL在背景上绘制旧对象，在前景上绘制新对象。

例如，假设我们向父对象添加了一个名为 button1 的按钮，然后又添加了另一个名为button2的按钮。 由于先创建了 button1，所以 button1 会被 button2 及其子对象覆盖。

![LVGL创建对象层级顺序](picture/lvgl笔记/20210401125003-1.png)

```c
/*Create a screen*/
lv_obj_t * scr = lv_obj_create(NULL, NULL);
lv_scr_load(scr);                                                               /*Load the screen*/

/*Create 2 buttons*/
lv_obj_t * btn1 = lv_btn_create(scr, NULL);         /*Create a button on the screen*/
lv_btn_set_fit(btn1, true, true);                   /*Enable to automatically set the size according to the content*/
lv_obj_set_pos(btn1, 60, 40);                                   /*Set the position of the button*/

lv_obj_t * btn2 = lv_btn_create(scr, btn1);         /*Copy the first button*/
lv_obj_set_pos(btn2, 180, 80);                          /*Set the position of the button*/

/*Add labels to the buttons*/
lv_obj_t * label1 = lv_label_create(btn1, NULL);        /*Create a label on the first button*/
lv_label_set_text(label1, "Button 1");                  /*Set the text of the label*/

lv_obj_t * label2 = lv_label_create(btn2, NULL);        /*Create a label on the second button*/
lv_label_set_text(label2, "Button 2");                  /*Set the text of the label*/

/*Delete the second label*/
lv_obj_del(label2);
```

#### LVGL将图层设到前台(foreground)展示

有几种方法可以将对象置于前台：

- 使用 `lv_obj_set_top(obj，true)` 。如果 obj 或它的任何子对象被点击，那么 LVGL 将自动将该对象带到前台。它的工作原理类似于PC机上典型的GUI，当点击背景中的窗口时，它会在前台展示。
- 使用lv_obj_move_foreground(obj) 显式地告诉库将对象带到前台。类似地，使用 `lv_obj_move_background(obj)` 将对象 obj 移动到背台。
- 当使用 `lv_obj_set_parent(obj，new_parent)` 时， obj 将在 `new_parent` 的前面。

#### LVGL顶层和系统层

LVGL 有两个特殊的图层； `layer_top` 和 `layer_sys` 。两者在显示器的所有屏幕上都是可见且通用的。 **但是，它们不会在多个物理显示器之间共享。** `layer_top` 始终位于默认屏幕 ( `lv_scr_act()` )的顶部， `layer_sys` 则位于 `layer_top` 的顶部。用户可以使用 `layer_top` 来创建一些随处可见的内容。例如，菜单栏，弹出窗口等。如果启用了 `click` 属性，那么 `layer_top` 将吸收所有用户单击并充当模态。

```c
lv_obj_set_click(lv_layer_top(), true);
```

`layer_sys` 也用于LVGL。例如，它将鼠标光标放在那里以确保它始终可见。

### 1.13 [LVGL 事件](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-events.html)

LVGL中可触发事件，用于与用户进行交互。例如一个对应对象的事件可以有：

- 被点击
- 被拖拽
- 被更改了数值
- 等等

我们可以将回调函数分配给对象以处理这些事件。例如：

```c
lv_obj_t * btn = lv_btn_create(lv_scr_act(), NULL);
lv_obj_set_event_cb(btn, my_event_cb);      /* 指定一个事件回调函数 */

    ...

    static void my_event_cb(lv_obj_t * obj, lv_event_t event)
    {
            switch(event) {
                    case LV_EVENT_PRESSED:              /* 对象被按下 */
                            printf("Pressed\n");
                            break;

                    case LV_EVENT_SHORT_CLICKED:        /* 对象被点击 */
                            printf("Short clicked\n");
                            break;

                    case LV_EVENT_CLICKED:              /* 对象被短点击 */
                            printf("Clicked\n");
                            break;

                    case LV_EVENT_LONG_PRESSED:         /* 对象被长按 */
                            printf("Long press\n");
                            break;

                    case LV_EVENT_LONG_PRESSED_REPEAT:  /* 对象被重复长按 */
                            printf("Long press repeat\n");
                            break;

                    case LV_EVENT_RELEASED:             /* 对象被释放 */
                            printf("Released\n");
                            break;
            }

               /*Etc.*/
    }
```

注意：多个对象可以使用同一回调函数。

#### LVGL事件类型

事件类型有如下几种：

通用事件

所有对象（例如 Buttons/Labels/Sliders 等）都可以接收这些通用事件。

与输入设备有关的事件

当用户按下/释放对象时，将发送这些消息。它们不仅用于指针，还可以用于键盘，编码器和按钮输入设备。访问输入设备概述部分以了解有关它们的更多信息。

- **LV_EVENT_PRESSED** 该对象被按下
- **LV_EVENT_PRESSING** 按下对象（按下时连续发送）
- **LV_EVENT_PRESS_LOST** 输入设备仍在按，但不再在对象上
- **LV_EVENT_SHORT_CLICKED** 在 `LV_INDEV_LONG_PRESS_TIME` 时间之前发布。如果拖动则不调用。
- **LV_EVENT_LONG_PRESSED** 按下 `LV_INDEV_LONG_PRESS_TIME` 时间。如果拖动则不调用。
- **LV_EVENT_LONG_PRESSED_REPEAT** 在每 `LV_INDEV_LONG_PRESS_REP_TIME` 毫秒的 `LV_INDEV_LONG_PRESS_TIME` 之后调用。如果拖动则不调用。
- **LV_EVENT_CLICKED** 如果未拖动则调用释放（无论长按）
- **LV_EVENT_RELEASED** 在上面每种情况下都被调用，即使对象已被拖动也被释放。如果在按下并从对象外部释放时从对象上滑出，则不会调用。在这种情况下，将发送 `LV_EVENT_PRESS_LOST` 。

指针相关的事件

这些事件仅由类似指针的输入设备（例如鼠标或触摸板）触发

- **LV_EVENT_DRAG_BEGIN** 开始拖动对象
- **LV_EVENT_DRAG_END** 拖动完成（包括拖动）
- **LV_EVENT_DRAG_THROW_BEGIN** 拖动开始（用“动量”拖动后释放）

与键盘和编码器相关的事件

这些事件由键盘和编码器输入设备发送。在[极客笔记](https://deepinout.com/)中的LVGL 输入设备部分中了解有关组的更多信息。

- **LV_EVENT_KEY** 键值发送到对象。通常在按下它或在长按之后重复时。可以通过以下方式检索键值 `uint32_t * key = lv_event_get_data()`
- **LV_EVENT_FOCUSED** 该对象集中在其组中
- **LV_EVENT_DEFOCUSED** 该对象在其组中散焦

一般事件

LVGL库发送的其他一般事件。

- **LV_EVENT_DELETE** 该对象正在被删除。释放相关的用户分配数据。

特殊事件

这些事件特定于特定的对象类型。

- **LV_EVENT_VALUE_CHANGED** 对象值已更改（例如，对于滑块）
- **LV_EVENT_INSERT** 有内容插入到对象中。 （通常到文本区域）
- **LV_EVENT_APPLY** 单击“确定”，“应用”或类似的特定按钮。 （通常来自键盘对象）
- **LV_EVENT_CANCEL** 单击“关闭”，“取消”或类似的特定按钮。 （通常来自键盘对象）
- **LV_EVENT_REFRESH** 查询以刷新对象。永远不会由库发送，但可以由用户发送。

请访问特定对象类型的文档，以了解对象类型使用了哪些事件。

#### LVGL自定义事件包含的数据

一些事件可能包含自定义数据。 例如，在某些情况下， `LV_EVENT_VALUE_CHANGED` 会告知新值。有关更多信息，请参见特定对象类型的文档。 要在事件回调中获取自定义数据，请使用 `lv_event_get_data()` 。

自定义数据的类型取决于发送对象，但如果是下面两种情况需要特殊对待：

- 数值，则为 uint32_t * 或 int32_t * 类型
- 字符，则为 char * 或 const char * 类型

#### LVGL手动发送事件

任意事件

要将事件手动发送到对象，请使用 `lv_event_send(obj, LV_EVENT_..., &custom_data)` 。

例如，它可以通过模拟按钮按下来手动关闭消息框（尽管有更简单的方法）：

```c
/*Simulate the press of the first button (indexes start from zero)*/
uint32_t btn_id = 0;
lv_event_send(mbox, LV_EVENT_VALUE_CHANGED, &btn_id);
```

刷新事件

`LV_EVENT_REFRESH` 是特殊事件，因为它旨在供用户用来通知对象刷新自身。一些例子：

- 通知标签根据一个或多个变量（例如当前时间）刷新其文本
- 语言更改时刷新标签
- 如果满足某些条件，请启用按钮（例如，输入正确的PIN）
- 如果超出限制，则向对象添加样式/从对象删除样式等

处理类似情况的最简单方法是利用以下函数：

`lv_event_send_refresh(obj)` 只是 `lv_event_send(obj, LV_EVENT_REFRESH, NULL)` 的包装。因此，它仅向对象发送 `LV_EVENT_REFRESH` 。

`lv_event_send_refresh_recursive(obj)` 将 `LV_EVENT_REFRESH` 事件发送给对象及其所有子对象。如果将 `NULL` 作为参数传递，则将刷新所有显示的所有对象。

### 1.14 [LVGL 样式](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-styles.html)

#### LVGL 样式简介

样式用于设置对象的外观。 lvgl 中的样式在很大程度上受到 CSS 的启发。简而言之，概念如下：

- 样式是 `lv_style_t` 变量，可以保存属性，例如边框宽度，文本颜色等。它类似于 CSS 中的类。
- 并非必须指定所有属性。未指定的属性将使用默认值。
- 可以将样式分配给对象以更改其外观。
- 样式可以被任意数量的对象使用。
- 样式可以级联，也就是可以将多个样式分配给一个对象，并且每种样式可以具有不同的属性。例如，style_btn 可能会导致默认的灰色按钮，并且 style_btn_red 只能添加一个 background-color=red 以覆盖背景色
- 后面添加的样式具有更高的优先级。这意味着，如果以两种样式指定属性，则将使用后面添加的样式。
- 如果未在当前对象中指定样式，则某些属性(例如，文本颜色)默认从父对象继承。
- 对象具有比“普通”样式更高优先级的局部样式。
- 与 CSS (伪类描述不同的状态，例如： hover )不同，在 lvgl 中，将属性分配给给定的状态。(即“类”与状态无关，但是每个属性都有一个状态)
- 当对象更改状态时可以使用转场过渡效果。

#### LVGL 样式的状态(States)

- **LV_STATE_DEFAULT** (0x00)：正常，已释放
- **LV_STATE_CHECKED** (0x01)：切换或选中
- **LV_STATE_FOCUSED** (0x02)：通过键盘或编码器聚焦或通过触摸板/鼠标单击
- **LV_STATE_EDITED** (0x04)：由编码器编辑
- **LV_STATE_HOVERED** (0x08)：鼠标悬停(现在不支持)
- **LV_STATE_PRESSED** (0x10)：已按下
- **LV_STATE_DISABLED** (0x20)：禁用或 void

可以将状态进行组合使用，如： `LV_STATE_FOCUSED | LV_STATE_PRESSED`

可以在每种状态和状态组合中定义样式的属性。例如，为默认和按下状态设置不同的背景颜色。如果未在状态中定义属性，则将使用最佳匹配状态的属性。通常，它表示带有 `LV_STATE_DEFAULT` 状态的属性。如果即使对于默认状态也未设置该属性，则将使用默认值。

但是，“最佳匹配状态的属性”到底是什么意思？优先级由其值显示(请阅读上面：”样式的状态(States)“)。 **值越大，优先级越高** 。 为了确定要使用哪个属性，我们举一个例子。让我们来看看背景色是怎样定义的：

- `LV_STATE_DEFAULT` : white
- `LV_STATE_PRESSED` : gray
- `LV_STATE_FOCUSED` : red

1. 默认情况下，对象处于默认状态，因此很简单：该属性在对象的当前状态中定义为白色
2. 按下对象时，有2个相关属性：默认为白色(默认与每个状态有关)和按下为灰色。按下状态的优先级为0x10，高于默认状态的0x00优先级，因此将使用灰色。
3. 当物体聚焦时，会发生与按下状态相同的事情，并且将使用红色。(焦点状态的优先级高于默认状态)。
4. 聚焦并按下对象时，灰色和红色都可以使用，但是按下状态的优先级高于聚焦，因此将使用灰色。
5. 可以为设置例如玫瑰色。在这种情况下，此组合状态的优先级为0x02 + 0x10 = 0x12，该优先级高于按下状态的优先级，因此将使用玫瑰色。LV_STATE_PRESSED | LV_STATE_FOCUSED
6. 选中对象后，没有属性可以设置此状态的背景色。因此，在缺少更好的选择的情况下，对象在默认状态的属性中仍为白色。

注意事项：

- 如果要为所有状态设置属性(例如红色背景色)，只需将其设置为默认状态即可。如果对象找不到其当前状态的属性，它将回退到默认状态的属性。
- 使用ORed状态来描述复杂情况的属性。(例如，按+选中+集中)
- 对不同的状态使用不同的样式元素可能是一个好主意。例如，很难找到释放，按下，选中+按下，聚焦，聚焦+按下，聚焦+按下+选中 等状态的背景颜色。相反，例如，将背景色用于按下和选中状态，并使用不同的边框颜色指示聚焦状态。

#### LVGL 级联样式

不需要将所有属性设置为一种样式。可以向对象添加更多样式，然后让后来添加的样式修改或扩展其他样式的属性。例如，创建常规的灰色按钮样式，并为仅设置新的背景色的红色按钮创建新的样式。

在CSS中，所有使用的类都像 `<div class=".btn .btn-red">` 一样列出，这是相同的概念。

**较晚添加的样式优先于较早的样式。** 因此，在上面的灰色/红色按钮示例中，应首先添加常规按钮样式，然后再添加红色样式。 但是，仍然要优先考虑优先级问题。因此，让我们研究以下情况：

- 基本的按钮样式定义了默认状态为深灰色和按下状态为浅灰色
- 红色按钮样式仅在默认状态下将背景色定义为红色

在这种情况下，释放按钮时(它处于默认状态)它将是红色的，因为在最后添加的样式(红色样式)中找到了完美的匹配。按下按钮时，浅灰色是更好的搭配，因为它完美地描述了当前状态，因此按钮将是浅灰色的。

#### LVGL 样式继承

某些属性(通常与文本相关)可以从父对象的样式继承。仅当未以对象的样式(即使在默认状态下)设置给定属性时，才应用继承。

在这种情况下，如果该属性是可继承的，则该属性的值也将在父级中搜索，直到一部分可以告诉该属性的值为止。父母会用自己的状态来说明价值。按下按钮后，文字颜色就从这里来，将使用按下的文字颜色。

#### LVGL 零件(Parts)的样式

对象可以具有自己样式的零件。例如，页面包含四个部分(Parts)：

- 背景(Background)
- 可卷动(Scrollable)
- 滚动条(Scrollable)
- 边缘闪光(Edge flash)

对象零件有三种类型：主要(main)，虚拟(virtual)和真实(real)。

主要(main)部分通常是对象的背景和最大部分。某些对象只有一个主要部分。例如，一个按钮只有一个背景。

虚拟(virtual)小部件是实时绘制到主体小部件的其他小部件。它们后面没有“真实”对象。例如，页面的滚动条不是真实的对象，仅在绘制页面背景时才绘制。虚拟小部件始终具有与主要小部件相同的状态。如果可以继承该属性，则在转到父级之前还将考虑主体部分。

真实(real)小部件是由主对象创建和管理的真实对象。例如，页面的可滚动部分是真实对象。实际小部件的状态可能与主要小部件的状态不同。

#### LVGL 初始化样式并设置、获取属性

样式存储在 `lv_style_t` 变量中。样式变量应为 **static** 、 **全局** 或 **动态分配** 。 也就是说，它们不能是函数中的局部变量，在函数存在时销毁。 在使用样式之前，应使用 `lv_style_init(&my_style)` 进行初始化 。 初始化后，我们就可以设置样式的属性。 属性集函数如下所示：`lv_style_set_<property_name>(&style, <state>, <value>);` 例如，上面提到的示例如下所示：

```c
static lv_style_t style1;
lv_style_init(&style1);
lv_style_set_size(&style1, LV_STATE_DEFAULT, 4);
lv_style_set_bg_opa(&style1, LV_STATE_DEFAULT, LV_OPA_COVER);
lv_style_set_bg_color(&style1, LV_STATE_DEFAULT, lv_color_hex3(0xeee));
lv_style_set_radius(&style1, LV_STATE_DEFAULT, LV_RADIUS_CIRCLE);
lv_style_set_pad_right(&style1, LV_STATE_DEFAULT, 4);
```

可以使用 lv_style_copy(＆style_destination，＆style_source) 复制样式。复制后，仍然可以自由添加属性。

要删除样式的属性，请使用：

```c
lv_style_remove_prop(&style, LV_STYLE_BG_COLOR | (LV_STATE_PRESSED << LV_STYLE_STATE_POS));
```

要从给定状态的样式中获取值，可以使用以下原型的功能：

```c
_lv_style_get_color/int/opa/ptr(&style, <prop>, <result buf>);
```

这样将选择最匹配的属性，并返回其优先级。如果找不到该属性，将返回 `-1` 。

函数的形式 (…color/int/opa/ptr) 应根据 `<prop>` 的类型使用。

例如：

```c
lv_color_t color;
int16_t res;
res = _lv_style_get_color(&style1,  LV_STYLE_BG_COLOR | (LV_STATE_PRESSED << LV_STYLE_STATE_POS), &color);
if(res >= 0) {
  //the bg_color is loaded into `color`
}
```

要重置样式(释放所有数据)，调用：

```c
lv_style_reset(&style);
```

#### LVGL 管理样式列表

样式本身不能发挥作用。只有将其分配给对象使用才能生效。对象的每个部分都存储一个样式列表，该列表是已分配样式的列表。

调用 `lv_obj_add_style(obj, <part>, &style);` 将样式添加到对象，例如：

```c
lv_obj_add_style(btn, LV_BTN_PART_MAIN, &btn);      /*Default button style*/
lv_obj_add_style(btn, LV_BTN_PART_MAIN, &btn_red);  /*Overwrite only a some colors to red*/
```

调用下面的函数重置对象样式列表：

```c
lv_obj_reset_style_list(obj, <part>);
```

如果已经分配给对象的样式发生更改(即，其属性之一设置为新值)，则应通过以下方式通知使用该样式的对象：

```c
lv_obj_refresh_style(obj)
```

要刷新所有零件和属性，调用：

```c
lv_obj_refresh_style(obj, LV_OBJ_PART_ALL, LV_STYLE_PROP_ALL)
```

要获取属性的最终值，包括级联，继承，局部样式和过渡(请参见下文)，可以使用以下类似的函数获取：

```c
lv_obj_get_style_<property_name>(obj, <part>);
```

这些函数使用对象的当前状态，如果没有更好的候选者，则返回默认值。例如：

```c
lv_color_t color = lv_obj_get_style_bg_color(btn, LV_BTN_PART_MAIN);
```

#### LVGL 本地风格

在对象的样式列表中，也可以存储所谓的局部属性。其与 CSS 的 `<div style="color:red">` 概念相同。局部样式与普通样式相同，但是它仅属于给定的对象， **不能与其他对象共享** 。要设置本地属性，请使用函数：

```c
lv_obj_set_style_local_<property_name>(obj, <part>, <state>, <value>);
```

例如

```c
lv_obj_set_style_local_bg_color(btn, LV_BTN_PART_MAIN, LV_STATE_DEFAULT, LV_COLOR_RED);
```

#### LVGL 样式过渡

默认情况下，当对象更改状态(例如，按下状态)时，会立即设置新状态下的新属性。但是，通过过渡，可以在状态改变时播放过渡动画。例如：在按下按钮后，可以在300毫秒内将其背景色设置为所按下的颜色。

过渡的参数存储在样式中。可以设置：

- 过渡时间
- 开始过渡之前的延迟
- 动画路径(也称为计时功能)
- 要设置动画的属性

可以为每个状态定义过渡属性。例如，将500 ms过渡时间设置为默认状态将意味着当对象进入默认状态时，将应用500 ms过渡时间。在按下状态下设置100 ms过渡时间将意味着在进入按下状态时100 ms过渡时间。因此，此示例配置将导致快速进入印刷机状态而缓慢回到默认状态。

#### LVGL 样式属性

样式中可以使用以下属性：

混合属性

- **radius(lv_style_int_t)**： 设置背景的半径。0：无半径，LV_RADIUS_CIRCLE：最大半径。默认值：0。
- **clip_corner(bool)**： 为true可以将溢出的内容剪切到圆角(半径> 0)上。默认值：false。
- **size(lv_style_int_t)**： 小部件内部元素的大小。是否使用此属性，请参见窗口小部件的文档。默认值：。LV_DPI / 20
- **transform_width (lv_style_int_t)**： 使用此值使对象在两侧更宽。默认值：0。
- **transform_height (lv_style_int_t)**：使用此值使对象在两侧都较高。默认值：0。
- **transform_angle (lv_style_int_t)**： 旋转类似图像的对象。它的uinit为0.1度，对于45度使用450。默认值：0。
- **transform_zoom (lv_style_int_t)**： 缩放类似图像的对象。LV_IMG_ZOOM_NONE正常大小为256(或)，一半为128，一半为512，等等。默认值：LV_IMG_ZOOM_NONE。
- **opa_scale(lv_style_int_t)**： 继承。按此比例缩小对象的所有不透明度值。由于继承了子对象，因此也会受到影响。默认值：LV_OPA_COVER。

#### 填充和边距属性

填充可在边缘的内侧设置空间。意思是“我不要我的孩子们离我的身体太近，所以要保留这个空间”。填充内部设置了孩子之间的“差距”。 边距在边缘的外侧设置空间。意思是“我想要我周围的空间”。 如果启用了布局或 自动调整，则这些属性通常由Container对象使用。但是，其他小部件也使用它们来设置间距。有关详细信息，请参见[极客笔记](https://deepinout.com/)中的 小部件(widgets) 的文档。

- **pad_top(lv_style_int_t)**： 在顶部设置填充。默认值：0。
- **pad_bottom(lv_style_int_t)**： 在底部设置填充。默认值：0。
- **pad_left(lv_style_int_t)**： 在左侧设置填充。默认值：0。
- **pad_right(lv_style_int_t)**： 在右侧设置填充。默认值：0。
- **pad_inner(lv_style_int_t)**： 设置子对象之间对象内部的填充。默认值：0。
- **margin_top(lv_style_int_t)**： 在顶部设置边距。默认值：0。
- **margin_bottom(lv_style_int_t)**： 在底部设置边距。默认值：0。
- **margin_left(lv_style_int_t)**： 在左边设置边距。默认值：0。
- **margin_right(lv_style_int_t)**： 在右边设置边距。默认值：0。

#### 背景属性

背景是一个简单的矩形，可以具有渐变和半径舍入。

- **bg_color** ( `lv_color_t` )： 指定背景的颜色。默认值：**LV_COLOR_WHITE**。
- **bg_opa** ( `lv_opa_t` )： 指定背景的不透明度。默认值：**LV_OPA_TRANSP**。
- **bg_grad_color** ( `lv_color_t` )： 指定背景渐变的颜色。右侧或底部的颜色是： **bg_grad_dir != LV_GRAD_DIR_NONE**。默认值： **LV_COLOR_WHITE**
- **bg_main_stop** ( `uint8_t` )： 指定渐变应从何处开始。0：最左/最上位置，255：最右/最下位置。默认值：0。
- **bg_grad_stop** ( `uint8_t` )： 指定渐变应在何处停止。0：最左/最上位置，255：最右/最下位置。预设值：255。
- **bg_grad_dir** ( `lv_grad_dir_t` )： 指定渐变的方向。可以是 **LV_GRAD_DIR_NONE/HOR/VER**。默认值：**LV_GRAD_DIR_NONE**。
- **bg_blend_mode** ( `lv_blend_mode_t` )： 将混合模式设置为背景。可以是 **LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE)**。默认值：**LV_BLEND_MODE_NORMAL**。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-1.png)

渐变的矩形背景

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the background style properties
 */
void lv_ex_style_1(void)
{
        static lv_style_t style;
        lv_style_init(&style);
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);

        /*Make a gradient*/
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);
        lv_style_set_bg_grad_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_bg_grad_dir(&style, LV_STATE_DEFAULT, LV_GRAD_DIR_VER);

        /*Shift the gradient to the bottom*/
        lv_style_set_bg_main_stop(&style, LV_STATE_DEFAULT, 128);
        lv_style_set_bg_grad_stop(&style, LV_STATE_DEFAULT, 192);


        /*Create an object with the new style*/
        lv_obj_t * obj = lv_obj_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_OBJ_PART_MAIN, &style);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
}
```

#### 边框属性

边框绘制在背景顶部。它具有圆角 `半径` 。

- **border_color** ( `lv_color_t` )： 指定边框的颜色。默认值：**LV_COLOR_BLACK**。
- **border_opa** ( `lv_opa_t` )： 指定边框的不透明度。默认值：**LV_OPA_COVER**。
- **border_width** ( `lv_style_int_t` )： 设置边框的宽度。默认值：**0**。
- **border_side** ( `lv_border_side_t` )： 指定要绘制边框的哪一侧。可以是 **LV_BORDER_SIDE_NONE/LEFT/RIGHT/TOP/BOTTOM/FULL** 。ORed值也是可能的。默认值： **LV_BORDER_SIDE_FULL** 。
- **border_post** ( `bool` )： 如果true在绘制完所有子级之后绘制边框。默认值：**false**。
- **border_blend_mode** ( `lv_blend_mode_t` )： 设置边框的混合模式。可以是 **LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE**。默认值： **LV_BLEND_MODE_NORMAL**

![LVGL 样式属性](picture/lvgl笔记/20210401130804-2.png)

设置圆角边框

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the border style properties
 */
void lv_ex_style_2(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        /*Set a background color and a radius*/
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 20);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);

        /*Add border to the bottom+right*/
        lv_style_set_border_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_border_width(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_border_opa(&style, LV_STATE_DEFAULT, LV_OPA_50);
        lv_style_set_border_side(&style, LV_STATE_DEFAULT, LV_BORDER_SIDE_BOTTOM | LV_BORDER_SIDE_RIGHT);

        /*Create an object with the new style*/
        lv_obj_t * obj = lv_obj_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_OBJ_PART_MAIN, &style);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
}
```

#### 轮廓属性

轮廓类似于边框，但绘制在对象外部。

- **outline_color** ( `lv_color_t` )：指定轮廓的颜色。默认值： **LV_COLOR_BLACK** 。
- **outline_opa** ( `lv_opa_t` )：指定轮廓的不透明度。默认值：**LV_OPA_COVER**。
- **outline_width** ( `lv_style_int_t` )：设置轮廓的宽度。默认值：**0**。
- **outline_pad** ( `lv_style_int_t` )：设置对象和轮廓之间的空间。默认值：**0**。
- **outline_blend_mode** ( `lv_blend_mode_t` )：设置轮廓的混合模式。可以是 **LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE**。默认值： **LV_BLEND_MODE_NORMAL** 。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-3.png)

设置轮廓

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the outline style properties
 */
void lv_ex_style_3(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        /*Set a background color and a radius*/
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);

        /*Add outline*/
        lv_style_set_outline_width(&style, LV_STATE_DEFAULT, 2);
        lv_style_set_outline_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_outline_pad(&style, LV_STATE_DEFAULT, 8);

        /*Create an object with the new style*/
        lv_obj_t * obj = lv_obj_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_OBJ_PART_MAIN, &style);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
}
```

#### 阴影属性

阴影是对象下方的模糊区域。

- **shadow_color** ( `lv_color_t` )：指定阴影的颜色。默认值： **LV_COLOR_BLACK**。
- **shadow_opa** ( `lv_opa_t` )：指定阴影的不透明度。默认值：**LV_OPA_TRANSP**。
- **shadow_width** ( `lv_style_int_t` )：设置轮廓的宽度(模糊大小)。默认值：**0**。
- **shadow_ofs_x** ( `lv_style_int_t` )：设置阴影的X偏移量。默认值：**0**。
- **shadow_ofs_y** ( `lv_style_int_t` )：设置阴影的Y偏移量。默认值：**0**。
- **shadow_spread** ( `lv_style_int_t` )：在每个方向上使阴影大于背景的值达到此值。默认值：**0**。
- **shadow_blend_mode** ( `lv_blend_mode_t` )：设置阴影的混合模式。可以是 **LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE** 。默认值：**LV_BLEND_MODE_NOR**

![LVGL 样式属性](picture/lvgl笔记/20210401130804-4.png)

设置阴影

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the Shadow style properties
 */
void lv_ex_style_4(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        /*Set a background color and a radius*/
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);

        /*Add a shadow*/
        lv_style_set_shadow_width(&style, LV_STATE_DEFAULT, 8);
        lv_style_set_shadow_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_shadow_ofs_x(&style, LV_STATE_DEFAULT, 10);
        lv_style_set_shadow_ofs_y(&style, LV_STATE_DEFAULT, 20);

        /*Create an object with the new style*/
        lv_obj_t * obj = lv_obj_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_OBJ_PART_MAIN, &style);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
}
```

#### 图案属性

图案是在背景中间绘制或重复以填充整个背景的图像(或符号)。

- **pattern_image** (`const void *`)：指向 **lv_img_dsc_t** 变量的指针，图像文件或符号的路径。默认值：**NULL** 。
- **pattern_opa** (`lv_opa_t`) ：指定图案的不透明度。默认值： **LV_OPA_COVER** 。
- **pattern_recolor** (`lv_color_t`) ：将此颜色混合到图案图像中。如果是符号(文本)，它将是文本颜色。默认值： **LV_COLOR_BLACK** 。
- **pattern_recolor_opa** (`lv_opa_t`) ：重新着色的强度。默认值： **LV_OPA_TRANSP** (不重新着色)。
- **pattern_repeat** (`bool`)： `true` 图案将作为马赛克重复。 **false** ：将图案放置在背景中间。默认值： **false** 。
- **pattern_blend_mode** (`lv_blend_mode_t`) ：设置图案的混合模式。可以是 **LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE**。默认值：**LV_BLEND_MODE_NORMAL**。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-5.png)

设置图案

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the pattern style properties
 */
void lv_ex_style_5(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        /*Set a background color and a radius*/
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);

        /*Add a repeating pattern*/
        lv_style_set_pattern_image(&style, LV_STATE_DEFAULT, LV_SYMBOL_OK);
        lv_style_set_pattern_recolor(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_pattern_opa(&style, LV_STATE_DEFAULT, LV_OPA_50);
        lv_style_set_pattern_repeat(&style, LV_STATE_DEFAULT, true);

        /*Create an object with the new style*/
        lv_obj_t * obj = lv_obj_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_OBJ_PART_MAIN, &style);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
}
```

#### 数值属性

值是绘制到背景的任意文本。它可以是创建标签对象的轻量级替代。

- **value_str** ( `const char *` )：指向要显示的文本的指针。仅保存指针！(不要将局部变量与 lv_style_set_value_str 一起使用，而应使用静态，全局或动态分配的数据)。默认值： **NULL**
- **value_color** ( `lv_color_t` )：文本的颜色。默认值：**LV_COLOR_BLACK**。
- **value_opa** ( `lv_opa_t` )：文本的不透明度。默认值：**LV_OPA_COVER**。
- **value_font** ( `const lv_font_t *` )**：指向文本字体的指针。默认值：**NULL**
- **value_letter_space** ( `lv_style_int_t` )：文本的字母空间。默认值：**0**。
- **value_line_space** ( `lv_style_int_t` )：文本的行距。默认值：**0**。
- **value_align** ( `lv_align_t` )：文本的对齐方式。可以是 **LV_ALIGN_…** 。默认值：**LV_ALIGN_CENTER**。
- **value_ofs_x** ( `lv_style_int_t` )：与路线原始位置的X偏移量。默认值：**0**。
- **value_ofs_y** ( `lv_style_int_t` )：从路线的原始位置偏移Y。默认值：**0**。
- **value_blend_mode** ( `lv_blend_mode_t` )：设置文本的混合模式。可以是 **LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE**。默认值：**LV_BLEND_MODE_NORMAL**。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-6.png)

设置图案

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the value style properties
 */
void lv_ex_style_6(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        /*Set a background color and a radius*/
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);

        /*Add a value text properties*/
        lv_style_set_value_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_value_align(&style, LV_STATE_DEFAULT, LV_ALIGN_IN_BOTTOM_RIGHT);
        lv_style_set_value_ofs_x(&style, LV_STATE_DEFAULT, 10);
        lv_style_set_value_ofs_y(&style, LV_STATE_DEFAULT, 10);

        /*Create an object with the new style*/
        lv_obj_t * obj = lv_obj_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_OBJ_PART_MAIN, &style);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);

        /*Add a value text to the local style. This way every object can have different text*/
        lv_obj_set_style_local_value_str(obj, LV_OBJ_PART_MAIN, LV_STATE_DEFAULT, "Text");
}
```

#### 文字属性

文本对象的属性。

- **text_color** ( `lv_color_t` )：文本的颜色。默认值：**LV_COLOR_BLACK** 。
- **text_opa** ( `lv_opa_t` )：文本的不透明度。默认值：**LV_OPA_COVER** 。
- **text_font** ( `const lv_font_t *` )：指向文本字体的指针。默认值：。**NULL**
- **text_letter_space** ( `lv_style_int_t` )：文本的字母空间。默认值：**0** 。
- **text_line_space** ( `lv_style_int_t` )：文本的行距。默认值：**0** 。
- **text_decor** ( `lv_text_decor_t` )：添加文字修饰。可以是 **LV_TEXT_DECOR_NONE/UNDERLINE/STRIKETHROUGH** 。默认值：**LV_TEXT_DECOR_NONE**。
- **text_sel_color** ( `lv_color_t` )：设置文本选择的背景色。默认值：**LV_COLOR_BLACK**
- **text_blend_mode** ( `lv_blend_mode_t` )：设置文本的混合模式。可以**LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE** 。默认值：**LV_BLEND_MODE_NORMAL**。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-7.png)

设置文字

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the text style properties
 */
void lv_ex_style_7(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);
        lv_style_set_border_width(&style, LV_STATE_DEFAULT, 2);
        lv_style_set_border_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);

        lv_style_set_pad_top(&style, LV_STATE_DEFAULT, 10);
        lv_style_set_pad_bottom(&style, LV_STATE_DEFAULT, 10);
        lv_style_set_pad_left(&style, LV_STATE_DEFAULT, 10);
        lv_style_set_pad_right(&style, LV_STATE_DEFAULT, 10);

        lv_style_set_text_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_text_letter_space(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_text_line_space(&style, LV_STATE_DEFAULT, 20);
        lv_style_set_text_decor(&style, LV_STATE_DEFAULT, LV_TEXT_DECOR_UNDERLINE);

        /*Create an object with the new style*/
        lv_obj_t * obj = lv_label_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_LABEL_PART_MAIN, &style);
        lv_label_set_text(obj, "Text of\n"
                                                        "a label");
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
}
```

#### 线属性

线的属性。

- **line_color** ( `lv_color_t` )：线条的颜色。默认值：**LV_COLOR_BLACK**
- **line_opa** ( `lv_opa_t` )：直线的不透明度。默认值：**LV_OPA_COVER**
- **line_width** ( `lv_style_int_t` )：线的宽度。默认值：**0**。
- **line_dash_width** ( `lv_style_int_t` )：破折号的宽度。仅对水平或垂直线绘制虚线。**0**：禁用破折号。默认值：**0**。
- **line_dash_gap** ( `lv_style_int_t` )：两条虚线之间的间隙。仅对水平或垂直线绘制虚线。**0**：禁用破折号。默认值：**0**。
- **line_rounded** ( `bool` )：**true**：绘制圆角的线尾。默认值：**false**。
- **line_blend_mode** ( `lv_blend_mode_t` )：设置线条的混合模式。可以是 **LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE**。默认值：**LV_BLEND_MODE_NORMAL**。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-8.png)

设置线条

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the line style properties
 */
void lv_ex_style_8(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        lv_style_set_line_color(&style, LV_STATE_DEFAULT, LV_COLOR_GRAY);
        lv_style_set_line_width(&style, LV_STATE_DEFAULT, 6);
        lv_style_set_line_rounded(&style, LV_STATE_DEFAULT, true);
#if LV_USE_LINE
        /*Create an object with the new style*/
        lv_obj_t * obj = lv_line_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_LINE_PART_MAIN, &style);

        static lv_point_t p[] = {{10, 30}, {30, 50}, {100, 0}};
        lv_line_set_points(obj, p, 3);

        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
#endif
}
```

#### 图象属性

图像的属性。

- **image_recolor** ( `lv_color_t` )：将此颜色混合到图案图像中。如果是符号(文本)，它将是文本颜色。默认值： **LV_COLOR_BLACK**
- **image_recolor_opa** ( `lv_opa_t` )：重新着色的强度。默认值：( **LV_OPA_TRANSP** 不重新着色)。默认值： **LV_OPA_TRANSP**
- **image_opa** ( `lv_opa_t` )：图像的不透明度。默认值：**LV_OPA_COVER**
- **image_blend_mode** ( `lv_blend_mode_t` )：设置图像的混合模式。可以是 **LV_BLEND_MODE_NORMAL/ADDITIVE/SUBTRACTIVE**。默认值：**LV_BLEND_MODE_NORMAL**。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-9.png)

设置图象

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the image style properties
 */
void lv_ex_style_9(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        /*Set a background color and a radius*/
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);
        lv_style_set_border_width(&style, LV_STATE_DEFAULT, 2);
        lv_style_set_border_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);

        lv_style_set_pad_top(&style, LV_STATE_DEFAULT, 10);
        lv_style_set_pad_bottom(&style, LV_STATE_DEFAULT, 10);
        lv_style_set_pad_left(&style, LV_STATE_DEFAULT, 10);
        lv_style_set_pad_right(&style, LV_STATE_DEFAULT, 10);

        lv_style_set_image_recolor(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_image_recolor_opa(&style, LV_STATE_DEFAULT, LV_OPA_50);

#if LV_USE_IMG
        /*Create an object with the new style*/
        lv_obj_t * obj = lv_img_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_IMG_PART_MAIN, &style);
        LV_IMG_DECLARE(img_cogwheel_argb);
        lv_img_set_src(obj, &img_cogwheel_argb);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
#endif
}
```

#### 转换属性

用于描述状态更改动画的属性。

- **transition_time** ( `lv_style_int_t` )：过渡时间。默认值：**0**。
- **transition_delay** ( `lv_style_int_t` )：转换前的延迟。默认值：**0**。
- **transition_prop_1** ( `property name` )：应在其上应用过渡的属性。将属性名称与大写字母一起使用，例如：**LV_STYLE_LV_STYLE_BG_COLOR**。默认值：**0(none)**。
- **transition_prop_2** ( `property name` )：与transition_1相同，只是另一个属性。默认值：**0(none)**。
- **transition_prop_3** ( `property name` )：与transition_1相同，只是另一个属性。默认值：**0(none)**。
- **transition_prop_4** ( `property name` )：与transition_1相同，只是另一个属性。默认值：**0(none)**。
- **transition_prop_5** ( `property name` )：与transition_1相同，只是另一个属性。默认值：**0(none)**。
- **transition_prop_6** ( `property name` )：与transition_1相同，只是另一个属性。默认值：**0(none)**。
- **transition_path** ( `lv_anim_path_t` )：过渡的动画路径(path)。(需要为静态或全局变量，因为仅保存了其指针)。默认值： **lv_anim_path_def** (线性路径)。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-10.png)

转换属性

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the transitions style properties
 */
void lv_ex_style_10(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        /*Set a background color and a radius*/
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);

        /*Set different background color in pressed state*/
        lv_style_set_bg_color(&style, LV_STATE_PRESSED, LV_COLOR_GRAY);

        /*Set different transition time in default and pressed state
         *fast press, slower revert to default*/
        lv_style_set_transition_time(&style, LV_STATE_DEFAULT, 500);
        lv_style_set_transition_time(&style, LV_STATE_PRESSED, 200);

        /*Small delay to make transition more visible*/
        lv_style_set_transition_delay(&style, LV_STATE_DEFAULT, 100);

        /*Add `bg_color` to transitioned properties*/
        lv_style_set_transition_prop_1(&style, LV_STATE_DEFAULT, LV_STYLE_BG_COLOR);

        /*Create an object with the new style*/
        lv_obj_t * obj = lv_obj_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_OBJ_PART_MAIN, &style);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
}
```

#### 比例属性

鳞片状元素的辅助属性。体重秤具有正常区域和末端区域。顾名思义，结束区域是标度的结束，可以是临界值或void值。正常区域在结束区域之前。两个区域可能具有不同的属性。

- **scale_grad_color** ( `lv_color_t` )：在正常区域中，在比例尺线上对该颜色进行渐变。默认值：**LV_COLOR_BLACK**。
- **scale_end_color** ( `lv_color_t` )：结束区域中刻度线的颜色。默认值：**LV_COLOR_BLACK**。
- **scale_width** ( `lv_style_int_t` )：比例尺的宽度。默认值：。默认值：**LV_DPI/8**
- **scale_border_width** ( `lv_style_int_t` )：在标准区域的比例尺外侧绘制的边框的宽度。默认值：**0**。
- **scale_end_border_width** ( `lv_style_int_t` )：在结束区域的刻度外侧上绘制边框的宽度。默认值：**0**。
- **scale_end_line_width** ( `lv_style_int_t` )：结束区域中比例线的宽度。默认值：**0**。

![LVGL 样式属性](picture/lvgl笔记/20210401130804-11.png)

轮盘比例效果

上述效果的示例代码：

```c
#include "../../lv_examples.h"

/**
 * Using the scale style properties
 */
void lv_ex_style_11(void)
{
        static lv_style_t style;
        lv_style_init(&style);

        /*Set a background color and a radius*/
        lv_style_set_radius(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_bg_opa(&style, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style, LV_STATE_DEFAULT, LV_COLOR_SILVER);

        /*Set some paddings*/
        lv_style_set_pad_inner(&style, LV_STATE_DEFAULT, 20);
        lv_style_set_pad_top(&style, LV_STATE_DEFAULT, 20);
        lv_style_set_pad_left(&style, LV_STATE_DEFAULT, 5);
        lv_style_set_pad_right(&style, LV_STATE_DEFAULT, 5);

        lv_style_set_scale_end_color(&style, LV_STATE_DEFAULT, LV_COLOR_RED);
        lv_style_set_line_color(&style, LV_STATE_DEFAULT, LV_COLOR_WHITE);
        lv_style_set_scale_grad_color(&style, LV_STATE_DEFAULT, LV_COLOR_BLUE);
        lv_style_set_line_width(&style, LV_STATE_DEFAULT, 2);
        lv_style_set_scale_end_line_width(&style, LV_STATE_DEFAULT, 4);
        lv_style_set_scale_end_border_width(&style, LV_STATE_DEFAULT, 4);

        /*Gauge has a needle but for simplicity its style is not initialized here*/
#if LV_USE_GAUGE
        /*Create an object with the new style*/
        lv_obj_t * obj = lv_gauge_create(lv_scr_act(), NULL);
        lv_obj_add_style(obj, LV_GAUGE_PART_MAIN, &style);
        lv_obj_align(obj, NULL, LV_ALIGN_CENTER, 0, 0);
#endif
}
```

在小部件的文档中，您将看到诸如“小部件使用典型的背景属性”之类的句子。 “典型背景”属性是：

- 背景
- 边境
- 大纲
- 阴影
- 模式
- 值

#### LVGL 主题

主题是样式的集合。始终有一个活动主题，在创建对象时会自动应用其样式。它为UI提供了默认外观，可以通过添加其他样式来对其进行修改。

默认的主题被设定在 `lv_conf.h` 与 `LV_THEME_...` 中定义。每个主题都具有以下属性：

- 原色
- 二次色
- 小字体
- 普通字体
- 字幕字体
- 标题字体
- 标志(特定于给定主题)

如何使用这些属性取决于主题。 有3个内置主题：

- **empty** 空：未添加默认样式
- **material** 材质：令人印象深刻的现代主题-单声道：用于黑白显示的简单黑白主题
- **template** 模板：一个非常简单的主题，可以将其复制以创建自定义主题

#### 扩展主题

内置主题可以通过自定义主题进行扩展。如果创建了自定义主题，则可以选择“基本主题”。基本主题的样式将添加到自定义主题之前。可以链接任何数量的主题。例如，材料主题->自定义主题->黑暗主题。

这是有关如何基于当前活动的内置主题创建自定义主题的示例。

```c
/*Get the current theme (e.g. material). It will be the base of the custom theme.*/
lv_theme_t * base_theme = lv_theme_get_act();

/*Initialize a custom theme*/
static lv_theme_t custom_theme;                         /*Declare a theme*/
lv_theme_copy(&custom_theme, )base_theme;               /*Initialize the custom theme from the base theme*/
lv_theme_set_apply_cb(&custom_theme, custom_apply_cb);  /*Set a custom theme apply callback*/
lv_theme_set_base(custom_theme, base_theme);            /*Set the base theme of the csutom theme*/

/*Initialize styles for the new theme*/
static lv_style_t style1;
lv_style_init(&style1);
lv_style_set_bg_color(&style1, LV_STATE_DEFAULT, custom_theme.color_primary);

...

/*Add a custom apply callback*/
static void custom_apply_cb(lv_theme_t * th, lv_obj_t * obj, lv_theme_style_t name)
{
        lv_style_list_t * list;

        switch(name) {
                case LV_THEME_BTN:
                        list = lv_obj_get_style_list(obj, LV_BTN_PART_MAIN);
                        _lv_style_list_add_style(list, &my_style);
                        break;
        }
}
```

#### LVGL 样式示例

![LVGL 样式属性](picture/lvgl笔记/20210401130804-12.png)

造型按钮

上述效果的示例代码：

```c
#include "../../lv_examples.h"


/**
 * Create styles from scratch for buttons.
 */
void lv_ex_get_started_2(void)
{
        static lv_style_t style_btn;
        static lv_style_t style_btn_red;

        /*Create a simple button style*/
        lv_style_init(&style_btn);
        lv_style_set_radius(&style_btn, LV_STATE_DEFAULT, 10);
        lv_style_set_bg_opa(&style_btn, LV_STATE_DEFAULT, LV_OPA_COVER);
        lv_style_set_bg_color(&style_btn, LV_STATE_DEFAULT, LV_COLOR_SILVER);
        lv_style_set_bg_grad_color(&style_btn, LV_STATE_DEFAULT, LV_COLOR_GRAY);
        lv_style_set_bg_grad_dir(&style_btn, LV_STATE_DEFAULT, LV_GRAD_DIR_VER);

        /*Swap the colors in pressed state*/
        lv_style_set_bg_color(&style_btn, LV_STATE_PRESSED, LV_COLOR_GRAY);
        lv_style_set_bg_grad_color(&style_btn, LV_STATE_PRESSED, LV_COLOR_SILVER);

        /*Add a border*/
        lv_style_set_border_color(&style_btn, LV_STATE_DEFAULT, LV_COLOR_WHITE);
        lv_style_set_border_opa(&style_btn, LV_STATE_DEFAULT, LV_OPA_70);
        lv_style_set_border_width(&style_btn, LV_STATE_DEFAULT, 2);

        /*Different border color in focused state*/
        lv_style_set_border_color(&style_btn, LV_STATE_FOCUSED, LV_COLOR_BLUE);
        lv_style_set_border_color(&style_btn, LV_STATE_FOCUSED | LV_STATE_PRESSED, LV_COLOR_NAVY);

        /*Set the text style*/
        lv_style_set_text_color(&style_btn, LV_STATE_DEFAULT, LV_COLOR_WHITE);

        /*Make the button smaller when pressed*/
        lv_style_set_transform_height(&style_btn, LV_STATE_PRESSED, -5);
        lv_style_set_transform_width(&style_btn, LV_STATE_PRESSED, -10);
#if LV_USE_ANIMATION
        /*Add a transition to the size change*/
        static lv_anim_path_t path;
        lv_anim_path_init(&path);
        lv_anim_path_set_cb(&path, lv_anim_path_overshoot);

        lv_style_set_transition_prop_1(&style_btn, LV_STATE_DEFAULT, LV_STYLE_TRANSFORM_HEIGHT);
        lv_style_set_transition_prop_2(&style_btn, LV_STATE_DEFAULT, LV_STYLE_TRANSFORM_WIDTH);
        lv_style_set_transition_time(&style_btn, LV_STATE_DEFAULT, 300);
        lv_style_set_transition_path(&style_btn, LV_STATE_DEFAULT, &path);
#endif

        /*Create a red style. Change only some colors.*/
        lv_style_init(&style_btn_red);
        lv_style_set_bg_color(&style_btn_red, LV_STATE_DEFAULT, LV_COLOR_RED);
        lv_style_set_bg_grad_color(&style_btn_red, LV_STATE_DEFAULT, LV_COLOR_MAROON);
        lv_style_set_bg_color(&style_btn_red, LV_STATE_PRESSED, LV_COLOR_MAROON);
        lv_style_set_bg_grad_color(&style_btn_red, LV_STATE_PRESSED, LV_COLOR_RED);
        lv_style_set_text_color(&style_btn_red, LV_STATE_DEFAULT, LV_COLOR_WHITE);
#if LV_USE_BTN
        /*Create buttons and use the new styles*/
        lv_obj_t * btn = lv_btn_create(lv_scr_act(), NULL);     /*Add a button the current screen*/
        lv_obj_set_pos(btn, 10, 10);                            /*Set its position*/
        lv_obj_set_size(btn, 120, 50);                          /*Set its size*/
```

### 1.15 [LVGL 输入设备](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-input-devices.html)

**输入设备(Input devices)**，在 LVGL 中输入设备，有下面几种类型：

- 指针式输入设备，如触摸板或鼠标
- 键盘，如普通键盘或简单的数字键盘
- 带有左/右转向和推入选项的编码器
- 外部硬件按钮，分配给屏幕上的特定点

在进一步阅读本文之前，请先阅读 [输入设备接口](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-input-device-interface.html) 的部分内容

#### 指针

指针输入设备可以具有光标。(通常是鼠标)

```c
...
lv_indev_t * mouse_indev = lv_indev_drv_register(&indev_drv);

LV_IMG_DECLARE(mouse_cursor_icon);                          /*Declare the image file.*/
lv_obj_t * cursor_obj =  lv_img_create(lv_scr_act(), NULL); /*Create an image object for the cursor */
lv_img_set_src(cursor_obj, &mouse_cursor_icon);             /*Set the image source*/
lv_indev_set_cursor(mouse_indev, cursor_obj);               /*Connect the image  object to the driver*/
```

请注意，光标对象应设置为 `lv_obj_set_click(cursor_obj, false)` 。对于图像，默认情况下禁用单击。

#### 键盘和编码器

我们可以使用键盘或编码器完全控制用户界面，而无需触摸板或鼠标。它的作用类似于PC上的TAB键，可以在应用程序或网页中选择元素。

组

想要通过键盘或编码器控制的对象需要添加到Group中。在每个组中，只有一个集中的对象可以接收按下的键或编码器的动作。例如，如果将文本区域作为焦点，并且在键盘上按了某个字母，则将发送键并将其插入到文本区域中。同样，如果将滑块聚焦，然后按向左或向右箭头，则滑块的值将被更改。

需要将输入设备与组关联。一台输入设备只能将键发送到一组，但一组也可以从多个输入设备接收数据。

要创建组，请使用 `lv_group_t * g = lv_group_create()` ，然后将对象添加到组中，请使用 `lv_group_add_obj(g, obj)` 。

要将组与输入设备关联，请使用 `lv_indev_set_group(indev, g)` ，其中 `indev` 是 `lv_indev_drv_register()` 的返回值

#### 按键

有一些预定义的键具有特殊含义：

- **LV_KEY_NEXT** 专注于下一个对象
- **LV_KEY_PREV** 专注于上一个对象
- **LV_KEY_ENTER** 触发器 `LV_EVENT_PRESSED/CLICKED/LONG_PRESSED` 等事件
- **LV_KEY_UP** 增加值或向上移动
- **LV_KEY_DOWN** 减小值或向下移动
- **LV_KEY_RIGHT** 增加值或向右移动
- **LV_KEY_LEFT** 减小值或向左移动
- **LV_KEY_ESC** 关闭或退出(例如，关闭下拉列表)
- **LV_KEY_DEL** 删除(例如，“ 文本”区域中右侧的字符)
- **LV_KEY_BACKSPACE** 删除左侧的字符(例如，在文本区域中)
- **LV_KEY_HOME** 转到开头/顶部(例如，在“ 文本”区域中)
- **LV_KEY_END** 转到末尾(例如，在“ 文本”区域中)

最重要的特殊键是 `LV_KEY_NEXT/PREV` ， `LV_KEY_ENTER` 和 `LV_KEY_UP/DOWN/LEFT/RIGHT` 。在 `read_cb` 函数中，应将某些键转换为这些特殊键，以便在组中导航并与所选对象进行交互。

通常，仅使用 `LV_KEY_LEFT/RIGHT` 就足够了，因为大多数对象都可以用它们完全控制。

对于编码器，应仅使用 `LV_KEY_LEFT` ， `LV_KEY_RIGHT` 和 `LV_KEY_ENTER` 。

#### 编辑和浏览模式

由于键盘有很多键，因此很容易在对象之间导航并使用键盘进行编辑。但是，编码器的“键”数量有限，因此很难使用默认选项进行导航。创建导航和编辑是为了避免编码器出现此问题。

在导航模式下，编码器 `LV_KEY_LEFT/RIGHT` 转换为 `LV_KEY_NEXT/PREV` 。因此，将通过旋转编码器选择下一个或上一个对象。按 `LV_KEY_ENTER` 将更改为编辑模式。

在“ 编辑”模式下， `LV_KEY_NEXT/PREV` 通常用于编辑对象。根据对象的类型，短按或长按可将其 `LV_KEY_ENTER` 更改回导航模式。通常，无法按下的对象(如Slider)会在短按时离开“ 编辑”模式。但是，对于具有短单击含义的对象(例如Button)，需要长按。

#### 样式

如果通过触摸板单击对象或通过编码器或键盘将其聚焦，则转到 `LV_STATE_FOCUSED` 。因此，将重点应用样式。

如果对象进入编辑模式，它将进入 `LV_STATE_FOCUSED|LV_STATE_EDITED` 状态，因此将显示这些样式属性。

### 1.16 [LVGL 显示](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-displays.html)

**显示(Displays)**，LVGL中显示的基本概念在 [显示接口](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-display-interface.html) 部分中进行了说明。因此，在进一步阅读之前，请先阅读 [显示接口](https://deepinout.com/lvgl-tutorials/lvgl-getting-started/lvgl-display-interface.html) 部分。

#### 多种显示支持

在LVGL中，可以有多个显示，每个显示都有自己的驱动程序和对象。唯一的限制是，每个显示器都必须具有相同的色深（如中所述 `LV_COLOR_DEPTH` ）。如果在这方面显示有所不同，则可以在驱动程序中将渲染的图像转换为正确的格式 `flush_cb` 。

创建更多的显示很容易：只需初始化更多的显示缓冲区并为每个显示注册另一个驱动程序。创建UI时，用于 `lv_disp_set_default(disp)` 告诉库在其上创建对象的显示。

为什么要多显示器支持？这里有些例子：

- 具有带有本地UI的“常规” TFT显示屏，并根据需要在VNC上创建“虚拟”屏幕。（需要添加 VNC 驱动程序）。
- 具有大型TFT显示屏和小型单色显示屏。
- 在大型仪器或技术中具有一些较小且简单的显示器。
- 有两个大型TFT显示屏：一个用于客户，一个用于店员。

#### 仅使用一个显示器

使用更多显示器可能很有用，但在大多数情况下，并非必需。因此，如果仅注册一个显示器，则整个多显示器概念将被完全隐藏。默认情况下，最后创建的（唯一的）显示用作默认设置。

`lv_scr_act()` , `lv_scr_load(scr)` , `lv_layer_top()` , `lv_layer_sys()` , `LV_HOR_RES` 以及 `LV_VER_RES` 始终应用在最后创建的（默认）屏幕上。 如果将 `NULL` 作为 `disp` 参数传递给显示相关功能，通常将使用默认显示。 例如。 `lv_disp_trig_activity(NULL)` 将在默认屏幕上触发用户活动。（请参见下面的非活动状态）。

#### 镜面展示

要将显示器的图像镜像到另一台显示器，则无需使用多显示器支持。只需将在 `drv.flush_cb` 中接收的缓冲区也转移到另一个显示器。

#### 分割影像

可以从较小的显示创建较大的显示。可以如下创建它：

1. 将显示器的分辨率设置为大显示器的分辨率。
2. 在中 `drv.flush_cb` ，截断并修改 `area` 每个显示的参数。
3. 将缓冲区的内容发送到带有截断区域的每个显示。

#### 屏幕

每个显示器都有每组屏幕和屏幕上的对象。

确保不要混淆显示和屏幕：

- **显示** 是绘制像素的物理硬件。
- **屏幕** 是与特定显示器关联的高级根对象。一个显示器可以具有与其关联的多个屏幕，但反之则不然。

屏幕可以视为没有父级的最高级别的容器。屏幕的大小始终等于其显示，屏幕的大小始终为(0;0)。 因此，无法更改屏幕坐标，即，不能在屏幕上使用 lv_obj_set_pos(), lv_obj_set_size() 或类似功能。

可以从任何对象类型创建屏幕，但是，两种最典型的类型是 `基础对象`_ 和 `图像`_ （用于创建墙纸）。

要创建屏幕，请使用 `lv_obj_t * scr = lv_<type>_create(NULL, copy)` 。复制可以是另一个屏幕来复制它。

要加载屏幕，请使用 `lv_scr_load(scr)` 。要获取活动屏幕，请使用 `lv_scr_act()` 。这些功能在默认显示屏上起作用。如果要指定要处理的显示器，请使用 `lv_disp_get_scr_act(disp)` 和 `lv_disp_load_sc(disp，scr)` 。屏幕也可以加载动画。在这里阅读更多。

可以使用`lv_obj_del(scr)`删除屏幕，但请确保不删除当前加载的屏幕。

#### 透明屏幕

通常，屏幕的不透明度是 `LV_OPA_COVER` 为其子级提供坚实的背景。如果不是这种情况（不透明度<100％），则显示器的背景颜色或图像将可见。有关更多详细信息，请参见显示背景部分。如果显示器的背景不透明性也不是，则 `LV_OPA_COVERLVGL` 不会绘制纯色背景。

此配置（透明屏幕和显示器）可用于创建OSD菜单，例如在其中将视频播放到下层，并在上层创建菜单。

为了处理透明显示器，LVGL需要使用特殊（较慢）的颜色混合算法，因此需要使用LV_COLOR_SCREEN_TRANSPn 启用此功能lv_conf.h。由于此模式在像素的Alpha通道上运行，因此也是必需的。32位颜色的Alpha通道在没有对象的情况下将为0，在有实体对象的情况下将为255。LV_COLOR_DEPTH = 32

总而言之，要启用透明屏幕和显示以创建类似于OSD菜单的UI：

- 在 `lv_conf.h` 中启用 `LV_COLOR_SCREEN_TRANSP`
- 请务必使用 `LV_COLOR_DEPTH 32`
- 将屏幕的不透明度设置为 `LV_OPA_TRANSP` 例如 `lv_obj_set_style_local_bg_opa(lv_scr_act(), LV_OBJMASK_PART_MAIN, LV_STATE_DEFAULT, LV_OPA_TRANSP)`
- 使用 `lv_disp_set_bg_opa(NULL, LV_OPA_TRANSP);` 将显示不透明度设置为 `LV_OPA_TRANSP`

#### 显示器功能

#### 不活跃

在每个显示器上测量用户的不活动状态。每次使用输入设备（如果与显示器关联）都被视为一项活动。要获取自上一次活动以来经过的时间，请使用 `lv_disp_get_inactive_time(disp)` 。如果传递了 `NULL` ，则所有显示（不是默认显示）将返回整体最小的不活动时间。

可以使用手动触发活动 `lv_disp_trig_activity(disp)` 。如果 `disp` 为 `NULL` ，则将使用默认屏幕（并非所有显示）。

#### 背景

每个显示都有背景色，背景图像和背景不透明度属性。当当前屏幕是透明的或未定位为覆盖整个显示器时，它们将变为可见。

背景色是填充显示的一种简单颜色。可以使用 `lv_disp_set_bg_color(disp，color)` 进行调整；

背景图像是用作墙纸的文件路径或指向 `lv_img_dsc_t` 变量（转换后的图像）的指针。可以使用 `lv_disp_set_bg_color(disp，＆my_img);` 进行设置。如果设置了背景图片（非 `NULL` ），则背景将不会填充 `bg_color`。

可以使用 `lv_disp_set_bg_opa(disp，opa)` 调整背景颜色或图像的不透明度。

这些函数的`disp` 参数可以为 `NULL` ，以将其引用到默认显示。

#### 色彩

颜色模块处理所有与颜色相关的功能，例如更改颜色深度，从十六进制代码创建颜色，在颜色深度之间转换，混合颜色等。

颜色模块定义了以下变量类型：

- **lv_color1_t** 存储单色。为了兼容性，它也具有R，G，B字段，但它们始终是相同的值（1个字节）
- **lv_color8_t** 用于存储8位颜色（1字节）的R（3位），G（3位），B（2位）的结构体。
- **lv_color16_t** 用于存储16位颜色（2字节）的R（5位），G（6位），B（5位）的结构体。
- **lv_color32_t** 用于存储24位颜色（4字节）的R（8位），G（8位），B（8位）的结构体。
- **lv_color_t** 等于 `lv_color1/8/16/24_t` 根据颜色深度设置
- **lv_color_int_t** `uint8_t` ， `uint16_t` 或 `uint32_t` 根据颜色深度设置。用于从纯数字构建颜色阵列。
- **lv_opa_t** 一种简单的 `uint8_t` 类型，用于描述不透明度。

`lv_color_t` ， `lv_color1_t` ， `lv_color8_t` ， `lv_color16_t` 和 `lv_color32_t` 类型具有四个字段：

- **ch.red** 红色通道
- **ch.green** 绿色通道
- **ch.blue** 蓝色通道
- **full** 红+绿+蓝为一个数字

通过将 `LV_COLOR_DEPTH` 定义设置为1（单色），8、16或32，可以在lv_conf.h中设置当前颜色深度。

#### 转换颜色

可以将一种颜色从当前颜色深度转换为另一种颜色。转换器函数以数字返回，因此必须使用 `完整` 字段：

```c
lv_color_t c;
c.red   = 0x38;
c.green = 0x70;
c.blue  = 0xCC;

lv_color1_t c1;
c1.full = lv_color_to1(c);      /*Return 1 for light colors, 0 for dark colors*/

lv_color8_t c8;
c8.full = lv_color_to8(c);      /*Give a 8 bit number with the converted color*/

lv_color16_t c16;
c16.full = lv_color_to16(c); /*Give a 16 bit number with the converted color*/

lv_color32_t c24;
c32.full = lv_color_to32(c);    /*Give a 32 bit number with the converted color*/
```

#### 交换16种颜色

可以在lv_conf.h中设置 `LV_COLOR_16_SWAP` 以交换RGB565颜色的字节。如果通过SPI等面向字节的接口发送16位颜色，则很有用。

由于16位数字以Little Endian格式存储（低位地址的低位字节），因此接口将首先发送低位字节。但是，显示通常首先需要较高的字节。字节顺序不匹配会导致色彩严重失真。

#### 创建和混合颜色

可以使用LV_COLOR_MAKE宏以当前颜色深度创建颜色。它使用3个参数（红色，绿色，蓝色）作为8位数字。例如，创建浅红色： `my_color = COLOR_MAKE(0xFF,0x80,0x80)` .

颜色也可以从十六进制代码创建： `my_color = lv_color_hex(0x288ACF)` 或 `my_color = lv_folro_hex3(0x28C)` .

可以混合两种颜色：`mixed_color = lv_color_mix(color1, color2, ratio)` 。比例可以是 0..255。 0 表示全彩色 2，255 表示全彩色 1。

也可以使用 `lv_color_hsv_to_rgb(hue，saturation，value)` 从HSV空间创建颜色。色相应在 0..360 范围内，饱和度和值应在 0..100 范围内。

#### 不透明度

为了描述不透明度，创建了 `lv_opa_t` 类型作为 `uint8_t` 的包装。还介绍了一些定义：

- **LV_OPA_TRANSP** 值：0，表示不透明度使颜色完全透明
- **LV_OPA_10** 值：25，表示颜色仅覆盖一点
- **LV_OPA_20 … OPA_80** 比较常用
- **LV_OPA_90** 值：229，表示几乎完全覆盖的颜色
- **LV_OPA_COVER** 值：255，表示颜色完全覆盖

还可以将 `LV_OPA_*` 定义 `lv_color_mix()` 作为比率使用。

#### 内置颜色

颜色模块定义了最基本的颜色，例如：

- \#FFFFFF LV_COLOR_WHITE
- \#000000 LV_COLOR_BLACK
- \#808080 LV_COLOR_GRAY
- \#c0c0c0 LV_COLOR_SILVER
- \#ff0000 LV_COLOR_RED
- \#800000 LV_COLOR_MAROON
- \#00ff00 LV_COLOR_LIME
- \#008000 LV_COLOR_GREEN
- \#808000 LV_COLOR_OLIVE
- \#0000ff LV_COLOR_BLUE
- \#000080 LV_COLOR_NAVY
- \#008080 LV_COLOR_TEAL
- \#00ffff LV_COLOR_CYAN
- \#00ffff LV_COLOR_AQUA
- \#800080 LV_COLOR_PURPLE
- \#ff00ff LV_COLOR_MAGENTA
- \#ffa500 LV_COLOR_ORANGE
- \#ffff00 LV_COLOR_YELLOW

以及 `LV_COLOR_WHITE` （全白）。