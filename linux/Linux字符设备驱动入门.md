# Linux字符设备驱动入门

## 字符设备简介

Linux内核驱动中分为**字符设备**、**块设备**和**网络设备**；其中字符设备是最常见的设备。字符设备是指能够像字节流一样被访问的设备，例如串口。

## Hello world

最简单的设备驱动示例

hello.c

```c
#include <linux/init.h>
#include <linux/module.h>

MODULE_LICENSE("GPL");


static int hello_init(void)
{
    printk("Hello world\n");
    return 0;
}

static void hello_exit(void)
{
    printk("Hello exit\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

Makefile

```makefile
obj-m += hello.o

all:
        make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
        make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

编译

```
$ make
make -C /lib/modules/4.15.0-194-generic/build M=/home/liangjunming/test_driver/driver modules
make[1]: Entering directory '/usr/src/linux-headers-4.15.0-194-generic'
  Building modules, stage 2.
  MODPOST 1 modules
make[1]: Leaving directory '/usr/src/linux-headers-4.15.0-194-generic'
```

使用insmod加载驱动

```
$ sudo insmod hello.ko
$ dmesg
[ 2424.345854] hello: loading out-of-tree module taints kernel.
[ 2424.345890] hello: module verification failed: signature and/or required key missing - tainting kernel
[ 2424.346415] Hello world
```

***dmesg**命令用来查看内核打印，`dmesg --clear` 清除内核打印*

使用 `rmmod` 卸载驱动

```
$ sudo rmmod hello
$ dmesg
[ 3173.816898] Hello exit
```

## 设备编号

字符设备通过文件系统中的特殊文件来存取，这些特殊文件称为**设备文件**或**设备节点**；惯例上它们位于 `/dev` 目录。这些特殊文件在使用 `ls -l` 命令输出的第一列有**"c"**标识

```
crw-rw-rw- 1 root root    1,  3 Dec 20 01:11 /dev/null
crw-rw-rw- 1 root root    1,  5 Dec 20 01:11 /dev/zero
crw--w---- 1 root tty     4,  0 Dec 20 01:11 /dev/tty0
crw--w---- 1 root tty     4,  1 Dec 20 01:11 /dev/tty1
```

另外可以看到两个用逗号隔开的数在最后修改日期前面，这里通常是文件长度出现的地方；这些数字是给**设备文件**的主次**设备编号**

传统上，**主编号**标识设备相连的驱动。例如：`/dev/null` 和 `/dev/zero` 都由驱动1来管理，而虚拟控制台和串口终端都由驱动4管理。**次编号**被内核用来决定引用那个设备

### 设备编号的内部表示

在内核中，在 `<linux/types.h>` 中定义的 `dev_t` 类型用来持有设备编号，主次部分都包括。

提取 `dev_t` 中的主或者次编号使用：

```c
MAJOR(dev_t dev);
MINOR(dev_t dev);
```

相反，如果你有主次编号，需要转换为一个 `dev_t`，使用：

```c
MKDEV(int major, int minor);
```

### 分配和释放设备号

在建立一个字符驱动时你的驱动需要做的第一件事是获取一个或多个设备编号来使用. 为 此目的的必要的函数是 **register_chrdev_region**, 在 **<linux/fs.h>** 中声明:

```c
int register_chrdev_region(dev_t first, unsigned int count, char *name);
```

这里，**first** 是你要分配的起始设备编号。**first** 的次编号部分常常是0，但是没有必须要求。**count** 是你请求的连续设备编号的总数。注意，如果 **count** 太大，你要求的范围可能溢出到下一个次编号；最后，**name** 是应当连接到这个编号范围的设备的名子；它会出现在 `/proc/devices` 和sysfs中

一个典型的 `/proc/devices` 文件看起来如下：

```
$ cat /proc/devices
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 ttyprintk
  7 vcs
 10 misc
 13 input
 21 sg
 29 fb
 89 i2c
108 ppp
180 usb

Block devices:
  7 loop
  8 sd
  9 md
 11 sr
 65 sd
 66 sd
```



如果你事先知道你需要哪个设备号，**register_chrdev_region** 工作的很好；但通常你不知道你的设备使用哪个主编号，所以在Linux内核开发中推荐使用 **alloc_chrdev_region** 来动态分配设备号

```c
int alloc_chrdev_region(dev_t *dev, unsigned int firstminor, unsigned int count, char *name);
```

这里 **dev** 是一个输出参数，它在函数执行成功后持有你分配范围的第一个设备号；**firstminor** 是请求的第一个要用的次编号，通常为0；**count** 和 **name** 参数和 **register_chrdev_region** 中的一样

不管使用哪个接口分配的设备编号，都应当在不使用它们的时候释放它；设备编号的释放使用：

``` c
void unregister_chrdev_region(dev_t first, unsigned int count);
```

## 字符设备注册

内核在内部使用定义在 `<linux/cdev.h>` 中的类型 `struct cdev` 来代表字符设备

使用 **cdev_alloc** 来分配一个这样的类型

``` c
struct cdev *my_cdev = cdev_alloc();
my_cdev->ops = &my_fops;
```

但是，偶尔你会想将 `cdev` 结构嵌入一个你自己自定义的结构中，这种情况可以使用 **cdev_init** 来初始化你已分配的结构

``` c
void cdev_init(struct cdev *cdev, struct file_operations *fops);
```

一旦 `cdev` 结构建立，最后的步骤是把它告诉内核，调用：

``` c
int cdev_add(struct cdev *dev, dev_t num, unsigned int count);
```

使用 `cdev_del` 删除一个字符设备：

``` c
void cdev_del(struct cdev *dev);
```

## 创建设备节点

在创建设备节点之前使用 `class_create` 创建一个class

``` c
dev->class = class_create(THIS_MODULE, "helloclass");
```

使用 `device_create` 创建设备节点

``` c
device = device_create(dev->class, NULL, MKDEV(dev->major, 0), NULL, "hello");
```

或者也可以使用命令 `mknod` 手动创建设备节点

## 完整例程

``` c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/types.h>
#include <linux/kdev_t.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>


MODULE_LICENSE("GPL");


#define DEV_COUNT 1
#define DEV_NAME "hellodev"

struct my_dev
{
    struct cdev c_dev;
    int major;
    dev_t devt;
    struct class *class;
};

struct my_dev my_dev;


int my_open(struct inode *inode, struct file *file)
{
    printk("%s\r\n", __func__);
    return 0;
}

int my_release(struct inode *inode, struct file *file)
{
    printk("%s\r\n", __func__);
    return 0;
}

ssize_t my_write(struct file *file, const char __user *usr, size_t size, loff_t *loff)
{
    int key_code;

    copy_from_user(&key_code, usr, sizeof(key_code));
    printk("key_code: %d \r\n", key_code);
    printk("%s\r\n", __func__);
    return 0;
}

struct file_operations fops =
{
    .open = my_open,
    .release = my_release,
    .write = my_write
};

static int dev_init(struct my_dev *dev)
{
    int ret = 0, error = 0;
    struct device *device = NULL;

    error = alloc_chrdev_region(&dev->devt, 0, DEV_COUNT, DEV_NAME);
    if (error < 0) {
        printk("alloc_chrdev_region error\r\n");
        ret = -EBUSY;
        goto fail;
    }
    printk("major = %d, minor = %d\r\n", MAJOR(dev->devt), MINOR(dev->devt));
    dev->major = MAJOR(dev->devt);

    cdev_init(&dev->c_dev, &fops);
    error = cdev_add(&dev->c_dev, dev->devt, DEV_COUNT);
    if (error < 0)
    {
        printk("cdev_add error\r\n");
        ret = -EBUSY;
        goto fail1;
    }

    dev->class = class_create(THIS_MODULE, "helloclass");
    if (IS_ERR(dev->class)) {
        printk("class_create error\r\n");
        ret = -EBUSY;
        goto fail2;
    }

    device = device_create(dev->class, NULL, MKDEV(dev->major, 0), NULL, "hello");
    if (device == NULL) {
        printk("device_create error\r\n");
        ret = -EBUSY;
        goto fail3;
    }

    return 0;

fail3:
    class_destroy(dev->class);
fail2:
    cdev_del(&dev->c_dev);
fail1:
    unregister_chrdev_region(dev->devt, DEV_COUNT);
fail:
    return ret;
}

static void dev_exit(struct my_dev *dev)
{
    device_destroy(dev->class, MKDEV(dev->major, 0));
    class_destroy(dev->class);
    cdev_del(&dev->c_dev);
    unregister_chrdev_region(dev->devt, DEV_COUNT);
}

static int __init hello_init(void)
{
    dev_init(&my_dev);
    printk("Hello module init\n");
    return 0;
}

static void __exit hello_exit(void)
{
    dev_exit(&my_dev);
    printk("Hello module exit\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

