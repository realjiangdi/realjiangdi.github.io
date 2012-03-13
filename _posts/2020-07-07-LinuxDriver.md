---
layout: post
title: Linux 驱动开发 学习笔记
tags: Linux驱动开发

---

Linux 设备驱动开发 学习记录

![bear](https://www.nationalgeographic.com/content/dam/animals/photos/000/006/667.ngsversion.1500322097297.adapt.1900.1.jpg)

<!--more-->

| 修订时间 | 修订内容                                                                       | 修订人 |
| :------- | :----------------------------------------------------------------------------- | :----- |
| 20200707 | 搭建文章框架；<br> 完成内核模块相关笔记及例程；<br> 完成字符驱动相关笔记及例程 | JD     |
| 20200708 | 修改字符设备驱动例程；                                                         | JD     |
| 20200721 | 完善字符设备驱动步骤及例程；                                                   | JD     |
| 20200725 | 补充Linux设备树信息及OF获取函数例程；                                          | JD     |
| 20200726 | 补充平台设备驱动的基本概念及例程；                                             | JD     |
| 20200727 | 完善平台设备驱动例程；                                                         | JD     |
| 20200813 | 完善高通4019设备树示例信息；                                                         | JD     |


学习目的：
- Be acquainted with the process of writing a device driver
- Be acquainted with the process of writing a kernel module

# Pre
## What is Driver ?
>趋势硬件设备工作 ---- 设备驱动

驱动通常完成如下具体工作：
- 读写设备寄存器
- 中断处理
- DMA通信
- 物理内存向虚拟内存映射


## 驱动设备分类
1. 字符设备 ----》》字符设备驱动    ----》》字符设备文件
2. 网络设备 ----》》网络设备驱动
3. 块设备   -----》》块设备驱动     ----》》块设备文件

驱动编写-》驱动编译-》驱动使用

### 字符设备
- I/O传输过程以字符为单位进行传输
- 用户对字符设备发出读/写请求时，实际的硬件及时发生读写操作

### 网络设备
- socket接口函数进行访问

### 块设备
- 数据传输以块（内存缓冲）为单位
- 用户操作与硬件操作不同步

## Linux内核模块
Linux内核抛弃将所有功能模块都编译到内核的做法，采用模块化的方法将各组件灵活添加和裁剪，并且驱动模块还可以动态加载、删除。

驱动的编写是基于模块。

模块编写：
+ 需包含如下头文件
```c
#include <linux/init.h>
#include <linux/module.h>
```
1. 模块加载
   1. `module_init(函数名);`
   ```c
   int __init xxx_func(void){

   }
   ```

2. 模块卸载
   1. `module_exit(函数名);`
   ```c
    void --exit xxx_func(void){

    }
   ```

3. GPL协议申明
   1. `MODULE_LICENSE("GPL");`

### Hello模块编写
```c
#include <linux/init.h>
#include <linux/module.h>

int __init demo_init(void){
    printk(KERN_INFO,"----%s---%s---%d---",__FILE__,__func__,__LINE__);
    return 0;
}

void __exit demo_exit(void){
 printk("----%s---%s---%d---",__FILE__,__func__,__LINE__);
}

module_init(demo_init);
module_exit(demo_exit);
MODULE_LICENSE("GPL");

```
### Linux 内核模块编译
编译器：gcc 若是在其他的平台的化，一般使用交叉编译工具。

编译内核模块的Makefile
1. 内部编译：将内核模块源文件放在内核源码中编译
2. 静态编译：将内核模块编译进uImage
3. 外部编译：将内核模块源文件放在内核源码外编译
4. 动态编译：编译生成动态模块xxx.ko

通常我们一般使用动态编译：
+ 内核编译方法：kernel/Documentation/kbuild/modules.txt

```makefile
.PHONY:clean
KERN_VERSION := $(shell uname -r)
KERNDIR := /lib/modules/$(KERN_VERSION)
PWD:= $(shell pwd)
obj-m:=demo.o

all:
   make -C $(KERNDIR) M=$(PWD) modules
clean：
   make -C $(KERNDIR) M=$(PWD) clean
```
查看.ko文件
```s
file demo.ko
```
### 内核模块的使用
装载内核模块的时候加载函数，并只执行一次。
```s
#查看内核模块信息
modinfo demo.ko
#列举当前所有的模块
lsmod
#将内核模块加载到内核中，再运行。
insmod demo.ko
modprobe demo.ko
# 查看内核日志信息
dmesg
# 清楚内核日志信息 clear
dmesg -c 
#将内核中的内核模块从内核中卸载
rmmod demo
```
事实上，`lsmod`命令实际读取并分析`/proc/modules`文件。


# 字符设备驱动
我们大多数设备都是都是字符驱动设备，所以我们的学习也以字符驱动设备为主。

字符驱动设备Example:LCD,鼠标，键盘，触摸屏

本节主要学习Linux字符设备驱动程序的结构和构造方法。
## 描述字符设备驱动的结构体
在Linux内核中，使用cdev结构体描述一个字符设备。cdev结构体包含**设备号`dev_t`**和**文件操作结构体`struct file_operations`**。
```c
struct cdev{
   struct module *owner;//THIS_MODULE
   const struct file_operations *ops;
   dev_t dev;  //设备号
   unsigned int count;  //设备个数
   struct list_head list;  
}
```

1. `dev_t` 设备号 
   + 表示设备的编号-具有唯一性
   + 32bit 无符号整型
   + 主设备号+次设备号（多个USB）
   + MAJOR(dev_t dev) //从设备号中提取主设备号 主设备由高12位组成
   + MINOR(dev_t dev)//从设备号中提取次设备号   得到低20位 `dev & (1U << 20)` 
   + MKDEV(int ma ,int mi)//根据主设备号生成次设备号
2. `struct file_operations` 操作方法集

   ```
   struct file_operations{
      struct module *owner;
      loff_t (*llseek)(struct file *,loff_t , int);
      ssize_t (*read)(struct file *,char __user *,size_t ,loff_t *);
      ssize_t (*write)(struct file*,const char __user*,size_t,loff_t *);
      unsigned int (*poll)(struct file *,struct poll_table_struct *);
      long(*unlocked_ioctl)(struct file *,struct vm_area_struct *);
      int (*open)(struct inode *,struct file *);
   }
   ```

## 编写字符设备驱动
字符设备驱动模块加载函数中，应该实现设备号的申请和cdev的注册，与此同时，在写在函数中，应该实现设备号的释放和cdev的注销。
1. 设备号（申请分配设备号或者指定设备号）
   1. 申请分配设备号`int alloc_chrdev_region(dev_t dev, unsigned baseminor,unigned count , const *name);`
   2. 注册指定设备号`int register_chrdev_region(dev_t from, unsigned count, char *name);`，但是使用注册指定设备号前，需要使用`MKDEV()`宏基于`major`来生成设备号。
   3. 注销字符设备之后，需要释放掉设备号`void unregister_chrdev_region(dev_t from,unsigned count)`
   4. 代码示例：
   
      ```c
      int major;
      int minor;
      dev_t devid;
      if(major){
            devid = MKDEV(major,0);
            register_chrdev_region(devid,1,"test");
      }
      else{
            alloc_chrdev_region(&devid,0,1,"test");
            major = MAJOR(devid);
            minor = MINOR(devid);
      }
      ```
2. 注册字符设备（字符设备cdev初始化`cdev_init` 和 向Linux系统注册字符设备`cdev_add`）
   1. `cdev`初始化函数`void cdev_init(struct cdev *cdev,const file_operations *fops);`
   2. `cdev`注册函数`int cdev_add(struct cdev*p,dev_t dev,unsigned count);`
   3. 卸载驱动时，需要将字符设备从Linux中注销`void cdev_del(struct cdev *p);`
3. 初始化字符设备文件操作函数集合(`file_operation`)
   1. 结合注册字符设备的代码示例如下所示：
   
   ```c
   struct cdev testcdev;
   static struct file_operations test_fops = {
      .owner = THIS_MODULE,
   };
   testcdev.owner = THIS_MODULE;
   cdev_init(&testcdev, &test_fops);
   cdev_add(&testcdev , devid ,1 );
   ```
4. 自动创建设备节点(加载模块成功后，会自动在/deb目录下创建对应的设备文件)
   1. 创建一个`Class`函数：`struct class *class_create(owner,name);`
   2. 创建一个设备`device`函数：`struct device *device_create(struct class *class, struct device *parent,dev_t devt,void *drvdata,const char *fmt);`
   3. 相应的，在卸载模块时，需要删除类和设备。
      1. `void class_destroy(struct class *cls);`
      2. `void device_destroy(struct class *class, dev_t devt);`
   4. 代码示例：
   
   ```c
   struct class *class;
   struct device *device;
   dev_t devid;

   class = class_create(THIS_MODULE, "xxx");
   device = device_create(class, NULL, devid,NULL ."xxx");

   ```


## 代码示例
源码示例
```c
#define Demo_RETURN_SUCCESS 0
#define Demo_RETURN_FAILED 1 
#define Demo_DEV_NAME "Demo"
#define Demo_CLASS_NAME "Demo_class"
#define Demo_COUNT 1

struct Demo_Dev {
    dev_t DemoDevId;
    struct cdev DemoCdev;
    struct class *DemoClass;
    struct device *DemoDevice;
    int major;
    int minor;
};

struct Demo_Dev DemoDev; 

static ssize_t Demo_read  (struct file *filp, char __user *buf, size_t cnt, loff_t * offet) {

    return Demo_RETURN_SUCCESS;
}
static ssize_t Demo_write (struct file *filp, const char __user *buf, size_t cnt, loff_t *offet){

    return Demo_RETURN_SUCCESS;
}
static int Demo_open (struct inode *inodep, struct file *filep){

    printk("%s %s line %d\r\n",__FILE__,__func__,__LINE__);

    return Demo_RETURN_SUCCESS;
}
static int Demo_release (struct inode *inodep, struct file *filep){
    
    printk("%s %s line %d\r\n",__FILE__,__func__,__LINE__);

    return Demo_RETURN_SUCCESS;
}

struct file_operations DemoFops ={
    .owner = THIS_MODULE,
    .open = Demo_open,
    .read = Demo_read,
    .write = Demo_write,
    .release = Demo_release,
};

static int __init DemoDev_init(void){
    int ret;
    printk("%s %s line %d\r\n",__FILE__,__func__,__LINE__);

    if(DemoDev.major){
        DemoDev.DemoDevId = MKDEV(DemoDev.major,0);
        if((ret = register_chrdev_region(DemoDev.DemoDevId,Demo_COUNT,Demo_DEV_NAME)) < 0)
            {
                printk("%s %s line %d\r\n",__FILE__,__func__,__LINE__);
                printk("faled to get dev_id\r\n");
                ret = Demo_RETURN_FAILED;
                goto err1;
            }
    }
    else{
        
        if((ret = alloc_chrdev_region(&DemoDev.DemoDevId,0,Demo_COUNT,Demo_DEV_NAME)) < 0)
            {
                printk("%s %s line %d\r\n",__FILE__,__func__,__LINE__);
                printk("faled to allocate dev_id\r\n");
                ret = Demo_RETURN_FAILED;
                goto err1;
            }
            DemoDev.major = MAJOR(DemoDev.DemoDevId);
            DemoDev.minor = MINOR(DemoDev.DemoDevId);
        printk("major id is %d, minor id is %d\r\n",DemoDev.major,DemoDev.minor);
    }

    DemoDev.DemoCdev.owner = THIS_MODULE;
    
    cdev_init(&DemoDev.DemoCdev,&DemoFops);
    cdev_add(&DemoDev.DemoCdev,DemoDev.DemoDevId,Demo_COUNT);

    DemoDev.DemoClass = class_create(THIS_MODULE,Demo_CLASS_NAME);
    if(IS_ERR(DemoDev.DemoClass)){
        ret = PTR_ERR(DemoDev.DemoClass);
        goto err2;
    }

    DemoDev.DemoDevice = device_create( DemoDev.DemoClass, NULL , DemoDev.DemoDevId, NULL ,Demo_DEV_NAME);
    if(IS_ERR(DemoDev.DemoDevice)){
        ret = PTR_ERR(DemoDev.DemoDevice);
        goto err3;
    }

    printk("HELLO Demo TEST DRIVER\r\n");
    return Demo_RETURN_SUCCESS;

err3:
    class_destroy(DemoDev.DemoClass);
err2:
    cdev_del(&DemoDev.DemoCdev);
    unregister_chrdev_region(DemoDev.DemoDevId,Demo_COUNT);
err1:
    printk("Demo TEST DRIVER FAILED\r\n");
    return ret;
}
static void __exit DemoDev_exit(void){
    printk("%s %s line %d\r\n",__FILE__,__func__,__LINE__);
    
    cdev_del(&DemoDev.DemoCdev);
    
    unregister_chrdev_region(DemoDev.DemoDevId,Demo_COUNT);
    
    device_destroy(DemoDev.DemoClass,DemoDev.DemoDevId);
    
    class_destroy(DemoDev.DemoClass);

    printk("BYE Demo TEST DRIVER\r\n");
}
module_init(DemoDev_init);
module_exit(DemoDev_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Jiang Di");
MODULE_DESCRIPTION("Demo Test Driver Module");

```


此外，在设备驱动执行读写操作时，用户空间不能直接访问内核空间的内存，反之亦同理。因此借助了两个函数完成如下操作：
```c
unsigned long copy_from_user(void *to, const void __user *from, unsigned long count);

unsigned long copy_to_user(void __user *to,const void *from,unsigned long count);

```
如果要复制的内存时简单的类型如`char、int、long` 等，可以使用简单`put_user()`和`get_user()`:
```c
int  val;
get_user(val, (int *)arg);

put_user(val,(int *)arg)
```
设置文件私有数据
```c
dev_t devid;
struct cdev cdev;
struct class *class;
struct device *device;
int major;
int minor;
/*
创建设备结构体
*/
struct test_dev{
   dev_t devid;
   struct cdev cdev;
   struct class *class;
   struct device *device;
   int major;
   int miror;
};
struct test_dev testdev;
static int test_open(struct inode *inode,struct file *filep){
   filp->private_data = &testdev;
}
```

# Linux设备树
## 什么是设备树
树形结构描述设备信息，如CPU数量，内存基地址等。
## 设备树的用法
用于实现**驱动代码和设备信息想分离**，这样即使硬件接口信息变化，但是驱动逻辑没有变化。驱动开发者只需要修改设备的文件信息，不需要重新修改/编写驱动代码。减轻了开发负担。

通常在ARM Linux中，一个`.dts`文件对应一个ARM的machine，一般放在`arch/arm/boot/dts`目录内。
### 基于高通IPQ4019 的设备树修改
想添加或修改的设备树节点，将位于`qca\src\linux-3.14\arch\arm\boot\dts\qcom-ipq40xx-ap.dk04.1.dtsi`下的设备树文件进行修改。

然后执行编译，将生成的内核版本升级到机器上。

内核升级方法：
1. 进入uboot；
2. 配置电脑主机为tftp的server；
3. 在终端侧ping电脑主机的ip地址；
4. ping通则执行内科升级命令`run upk`；
5. 输入`reset`命令执行重启操作；

## DTS DTB和 DTC的关系
- DTS: 设备树源码
- DTC: 设备书编译工具
- DTB: 设备树可执行文件

## DTS 基本语法
主要是熟悉一些常见的DTS基本语法，且可以进行适当的修改；

设备树的信息在/proc/device-tree可以查看。

### GPIO 设备树示例
基于高通IPQ4019
#### GPIO As Input
基于EC11 旋钮 设备树修改
```json
/*Configure GPIO 52 , 53 for knob functionality by adding the below node under pinctrl node.*/
			knob_0_pins: knob0_pinmux {
				mux {
					pins = "gpio52","gpio53";
					input-enable;
					bias-pull-up;
				};
			};
/*Then add knob pinctl node to knob parent nodes as follow:*/
		gpio_knobs {
			compatible = "gpio-knob";
			knob_a = <&tlmm 52 0>;
			knob_b = <&tlmm 53 0>;
			status = "okay";
		};
```
#### SPI 
基于OLED SSD1309 设备树修改
```json
		spi_1_pins: spi_1_pinmux {
				mux {
					pins =  "gpio13", "gpio14", "gpio15";
					function = "blsp_spi1";
					bias-disable;
				};
            cs1 {
               pins = "gpio12";
               function = "blsp_spi0";
            };
            cs2 {
               pins = "gpio45";
               function = "blsp_spi0";
            };
            dc {
               pins = "gpio5";
               function = "gpio";
               output-high;
            };
            reset {
               pins = "gpio60";
               function = "gpio";
               output-high;
            };
			};

      spi_0: spi@78b5000 { /* BLSP1 QUP1 */
			pinctrl-0 = <&spi_1_pins>;
			pinctrl-names = "default";
			status = "ok";

			m25p80@0 {
				#address-cells = <1>;
				#size-cells = <1>;
				reg = <0>;
				compatible = "n25q128a11";
				linux,modalias = "m25p80", "n25q128a11";
				spi-max-frequency = <24000000>;
				use-default-sizes;
			};

         ssd1309@1 {
            reg = <1>;
            compatible = "ssd1309";
            spi-max-frequency = <14000000>;/* 最初这里配置了24Mhz，但是oled不显示，后来调到14Mhz，就可以了*/
         };
		};
```
#### I2C
```json
			i2c_0_pins: i2c_0_pinmux {
				mux {
					pins = "gpio58", "gpio59";
					function = "blsp_i2c0";
					bias-disable;
				};
			};



		i2c_0: i2c@78b7000 { /* BLSP1 QUP2 */
			pinctrl-0 = <&i2c_0_pins>;
			pinctrl-1 = <&i2c_0_pins>;
			pinctrl-names = "i2c_active", "i2c_sleep";
			status = "ok";

			qca_codec: qca_codec@12 {
				compatible = "qca,ipq40xx-codec";
				reg = <0x12>;
				status = "disabled";
			};
			max17205: max17205@36 {
				compatible = "maxim,max17205";
				reg = <0x36>;
				status = "ok";
			};
			lcd_ts: lcd_ts@40 {
				compatible = "qca,gsl1680_ts";
				reg = <0x40>;
				status = "disabled";
			};
		};

```
## OF函数的使用
在驱动程序中调用OF函数来获取设备树中设备节点的相关信息。通常首先依据调用of_find_node_path在设备树中找到节点。
下面的例程简单介绍了如果找到对应设备节点，并获取设备节点的字符串，数组等信息。
```c
struct device_node *nd;
struct property *ppro;
char *str;
u32 def_value;

nd = of_find_node_by_path("/");
if(nd  == NULL){
   ret = -EINVAL;
   goto fail_findnd;
}

ppro = of_find_property(nd,"compatible",NULL);
if(ppro  == NULL){
   ret = -EINVAL;
   goto fail_findpro;
}
else{

   printk("compitable value is %s",ppro->values);
}

ret = of_property_read_string(nd,"status",str);
if(ret < 0){
   goto fail_rs;
}

ret = of_property_read_u32(nd,"",&def_value);
if(ret < 0){
   goto faile_read32;
}
   return 0;


fail_read32:
fail_rs:
fail_findppro:
fail_findnd:
   return ret;
```

## 设备树参考资料
- [Device Tree Usage](https://elinux.org/Device_Tree_Usage)
- [小白带你探索Linux设备树](https://blog.csdn.net/qq_35712169/article/details/104208420)
- [Documentation/dts-format.txt](https://git.kernel.org/pub/scm/utils/dtc/dtc.git/plain/Documentation/dts-format.txt?id=HEAD)

# Linux中断
Linux内核中断处理程序可以分解为两个半部：顶半部（Top Half）和底半部（Bottom Half）。

> Tips: 在Linux中，查看/proc/interrupts 文件可以获取系统中断的统计信息

## Linux中断编程
### 申请中断
通常在Linux中，使用中断的设备需要申请和释放对应的中断，具体编程操作如下所示：
```c
/*request irq*/
int request_irq(unsigned int irq, irq_handler_t handler,unsigned long flags, const char *name , void *dev);

/*free irq*/
void free_irq(unsigned int irp, void *dev_id);
```
- `irq`  申请的中断号
- `handler` 中断处理函数。中断发生时，系统调用这个函数
- `irqflags` 中断的触发方式和处理方式
- `name`
- `dev` 通常时Null或者这个设备的结构体
### tasklet 机制

```c

void xxx_do_tasklet(unsigned long);
DECLARE_TASKLET(xxx_tasklet , xxx_do_tasklet , 0);

void xxx_do_tasklet(unsigned long){

}
/* */
irqreturn_t xxx_interrupt(int irq, void *dev_id){
      tasklet_schedule(&xxx_tasklet);
}

int __init xxx_init(void){

   result = request_irq(xxx_irq, xxx_interrupt,0,"xxx",NULL);

   return IRQ_HANDLED;
}
void __exit xxx_exit(void){

   free_irq(xxx_irq,xxx_interrupt);
}
```

### 工作队列 机制
通常先定义一个工作队列和工作队列的处理函数，然后再模块初始化过程中，初始化工作队列并将其与处理函数绑定。
```c
struct work_struct xxx_wq;
void xxx_do_work(struct work_struct *work);

void xxx_do_work(struct work_struct *work){

}
irqreturn_t xxx_interrupt(int irq , void *dev_id){
   schedule_work(&xxx_wq);

   return IRQ_HANDLED;
}
__init int xxx_init(void){
   result = request_irq(xxx_irq,xxx_interrupt,0,"xxx",NULL);

   INIT_WORK(&xxx_wq,xxx_do_work);

}

__exit void xxx_exit(void){
   free_irq(xxx_irq,xxx_interrupt);
}

```
# 平台设备驱动
## 总线、设备与驱动
总线将设备与驱动绑定。再系统每注册一个设备，总线会与之匹配驱动。反之亦成立。
- 相应的平台设备`platform_device`结构体
- 相应的平台设备驱动`platfrom_driver`结构体
  - 平台设备驱动包含`probe()`,`remove`,`device_driver`实例等
- 相应的设备驱动`device_driver`结构体

基于`platform_devie` 与 `platform_driver` 的代码示例
```c
static int xxx_probe(struct platform_devie *pdev){

}
static int xxx_remove(struct platform_device *pdev){

}
static struct platform_driver xxx_driver = {
   .driver = {
      .name = "xxx"
      .owner = THIS_MODULE,
   }
   .probe = xxx_probe,
   .remove = xxx_remove,
};
module_platform_driver(xxx_driver);

/**/
static struct platform_device xxx_device = {
   .name = "xxx",
   .id   = -1.
};
```
### 平台设备资源和数据的获取
再`platform_device`结构体中 定义了设备的资源`resource`.通常，`resource`定义在BSP的板文件中。所以在具体的驱动中，可以调用`get_resource`这样的API来获取。Linux在3.x版本以后引入了设备树，故还可以从设备树中提取。
```c
struct resource {
   resource_size_t start;
   resource_size_t end;
   const char *name;
   unsigned long flags;
   struct resource *parent , *sibling, *child;
}
```
### 输入设备驱动

### Linxux SPI驱动框架

### Linux IIC驱动框架

### Linux INPUT子系统
input_dev 注册流程
```c
struct input_dev *inputdev;
static int __init xxx_init(void){
   inputdev = input_allocate_device();
   inputdev->name = "test_inputdev";
   /*设置时间和时间值*/
   /*ops*/
   /******/


   input_register_device(inputdev);
   return 0;
}
static void __exit xxx_exit(void){
   input_unregister_device(inputdev);
   input_free_device(inputdev);
}
```
上报输入事件 
```c
input_events(struct input_dev *dev, unsigned type , unsigned int code,int value);
/*
type  -  上报的事件类型(EV_KEY)  u16
code  -  事件码（KEY_0 KEY1 KEY_2) u16
value -  事件值( 1 /0)  u32
*/
input_sync(struct input_dev *dev);
```
与驱动模块想对应，用户态则需要做如下修改
```c
#inlucde <linux/input.h>
static struct input_event inputevent;

int fd;
fd = open("filename",O_RDWD);
while(1){
   err = read(fd, &inputevent, sizeof(inputevent));
   if(err > 0){
      switch (inputevent.type){
         case EV_KEY:
            if(inputevent.code < BTN_MISC){
               printf("key %d %s \r\n",inputevent.code,inputevent.value ? "perss" : "release");
            }
            else {
               printf("button %d %s \r\n" inputevent.code,inputevent.value ? "perss : "release");
            }
            break;

            case EV_REL:
               break;
            case EV_ABS :
               break;
            case EV_MSC :
               break;
      }
   }
   else {
      printf("failed read\r\n");
   }
   return 0;
}
```
#### Linux 自带案件驱动程序的使用
使用Linux内核自带的按键驱动程序很简单，只需要根据`Documentation/devicetree/bindings/input/gpio-keys.txt`这个文件在设备树中添加指定的设备节点即可。
### Linux MISC驱动
采用`misc`设备驱动可以简化字符设备驱动的编写，我们可以向Linux注册一个`miscdevice`设备，`miscdevice`是一个结构体，定义在头文件`linux/miscdevice.h`中。

注册MISC设备驱动代码示例：
```c
#define MISCxxx_Name    "xxx"

#define MISCxxx_MINOR   x

static int xxxx_open(struct inode *inode, struct file *filp){
   return 0;
}

static ssize_t xxxx_write(struct file *filp,const char __user *buf, size_t cnt, loff_t *offt){
   return 0;
}

static struct file_operations miscxxx_fops = {
   .owner = THIS_MODULE,
   .open = miscxxx_open,
   .write = miscxxx_write,
}
static struct miscdevice xxx_miscdev =  {
   .minor = MISCxxx_MINOR,
   .name = MISCxxx_NAME,
   .fops = &miscxxx_fops,
}
static int miscxxx_probe(struct platform_device *dev){
   int ret = 0;

   ret = misc_register(xxx_miscdev);
   if(ret < 0){
      printk("misc device register failed\r\n");
      return -EFAULT;
   }

   return 0;
}
static int miscxxx_remove(struct platform_device *dev){
   misc_deregister(&xxx_miscdev);
   return 0;
}
static const struct of_device_id xxx_of_match[] = {
   {  .compatible = "xxxxxxxx"},
   {  /* Sentinel */},
}

static struct platform_driver xxx_driver = {
   .driver = {
      .name = "driver_xxxx",
      .of_match_table = xxx_of_match,
   },
   .probe = miscxxx_probe,
   .remobe = miscxxx_remove,
}
static int __init miscxxx_init(void){
   
   return platform_driver_register(&xxx_driver);

}
static void __exit miscxxx_exit(void){
   platform_driver_unregister(&xxx_driver);
}

module_init(miscxxx_init);
module_exit(miscxxx_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("JiangDi");
```
所有的`misc`设备都属于同一个类，都在`/sys/class/misc`目录下。

驱动与设备匹配成功后，会生成/dev/miscxxx这个设备驱动文件。`ls /dev/miscxxx -l` 查看

### Linux I2C驱动
I2C 驱动注册流程
```c
static int xxx_probe(struct i2c_client *client, const struct i2c_device_id *id)
{
   /*ops*/

   return 0;
}

static int xxx_remove(struct i2c_client *client){
   /*ops*/

   retrun 0;
}

static const struct i2c_device_id xxx_id[] = {
   {"xxx",0},
   {  }
}
static const struct of_device_id xxx_of_match[] = {
   {  .compatible = ""},
   {  /* Sentinel */}
}
static struct i2c_driver xxx_driver = {
   .probe = xxx_probe,
   .remove = xxx_remove,
   .driver = {
      .owner = THIS_MODULE,
      .name = "xxx",
      .of_match_table = xxx_of_match,
   },
   .id_table = xxx_id,
}

static int __init xxx_init(void){
   int ret = 0;

   ret = i2c_add_driver(&xxx_driver);

   return ret;
}
static void __exit xxx_exit(void){
   i2c_del_driver(&xxx_driver);
}
module_init(xxx_init);
module_exit(xxx_exit);
```
### Linux SPI驱动
spi_driver 驱动注册示例
```c
static int xxx_probe(struct spi_device *spi){
   return 0;
}

static int xxx_remove(struct spi_device *spi){

   /*ops*/

   return 0;
}

static const struct spi_device_id xxx_id[] = {
   {"xxx",0},
   {}

};
static const struct of_device_id xxx_of_match [] = {
   {.compatible = "xxxx"},
   {  /* Sentinel */}
};

static struct spi_driver xxx_driver = {
   .probe = xxx_probem,
   .remove = xxx_remove,
   .driver = {
      .owner = THIS_MODULE,
      .name = "xxx",
      .of_match_table = xxx_of_match,
   },
   .id_table = xxx_id,
};

static int __init xxx_init(void){
   return spi_register_driver(&xxx_driver);

}

static void __exit xxx_exit(void){
   spi_unregister_driver(&xxx_driver);
}
module_init(xxx_init);
module_exit(xxx_exit);
```
#### SPI 驱动框架 参考资料
- [Linux SPI 驱动分析（1）— 结构框架](https://blog.csdn.net/zhoutaopower/article/details/99866773)


# Linux 阻塞与非阻塞
## 阻塞
## 非阻塞
### select 函数非阻塞
程序代码实例
```c
void main(void){
   int ret , fd;
   fd_set readfds;
   struct timeval teimout;
   fd = open("dev_xxx", O_RDWR | O_NONBLOCK);
   FD_ZERO(&readfds);
   FD_SET(fd, &readfds);

   timeout.tv_set  = 0;
   timeout.tv_usec = 500000;

   ret = select (fd + 1 ， &readfds,NULL,NULL,&timeout);
   switch(ret){
      case 0 :
         printf("timeout \r\n");
         break;
      case -1 :
         printf("error\r\n");
      default :
         if(FD_ISSET(fd,&readfds)){
            /*ops*/
         }
         break;
   }
}
```
### poll 函数非阻塞
程序代码示例
```c
void main(void){
   int ret;
   int fd;
   struct pollfd fds;

   fd = open(filename , O_RDWR | O_NONBLOCK);
   fds.fd = fd;
   fds.events = POLLIN;
   ret = poll(&fds , 1 ,500);
   if(ret){
      /*read ops*/
   }
   else if(ret == 0){
      /* time out*/
   }
   else if(ret < 0){
      /* error */
   }
}
```
### epoll 函数
代码示例
```c

```
## Linux异步通知

# Q&A
+ `register_chrdev_region`与`register_chrdev`和`alloc_chrdev_region`的区别？
  + 旧的字符设备号注册方式`register_chrdev`
    + 须实现确定好主设备号，且浪费大量次设备号。
  + 新的字符设备号注册方式`alloc_chrdev_region`和`register_chrdev_region`
    + 新的字符设备号注册方式就是为了解决这个问题。
  + 新的字符设备号注销函数`unregister_chrdev_region`
+ [Linux使用open函数无法打开驱动](https://blog.csdn.net/qq_43039392/article/details/106565740)


# Reference
- [【千锋】Linux驱动开发教程（程序员必备/2019版）](https://www.bilibili.com/video/av81769961)
- [嵌入式Linux驱动开发](https://d1.amobbs.com/bbs_upload782111/files_35/ourdev_600321OFQOPJ.pdf)
- [linux驱动学习笔记（linux驱动头文件说明）](https://blog.csdn.net/wanghanjiett/article/details/6791593)
- [Linux设备驱动开发详解-基于最新的Linux 4.0内核](https://book.douban.com/subject/26600201/)
- [Linux驱动开发](https://www.cnblogs.com/xiaojiang1025/category/918665.html)
- [SDM450设备树学习](https://github.com/SuperTao/LinuxDTS)
- [Linux设备驱动之Ioctl控制](https://www.cnblogs.com/geneil/archive/2011/12/04/2275372.html)
- [Linux设备驱动程序 之 ioctl](https://www.cnblogs.com/wanpengcoder/p/11760785.html)