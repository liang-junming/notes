# autoconf&automake

## 创建autoconf&automake工程

1. 准备源码结构如下：

   ```shell
   .
   └── src
       └── hello.c
   ```

2. 创建Makefile.am文件

   automake根据Makefile.am文件生成Makefile.in文件，后续configure脚本根据Makefile.in文件生成Makefile文件；也就是我们本来要编写Makefile，变成编写Makefile.am

   这里需要项目根目录下和src目录下的共两个Makefile.am文件

   **Makefile.am**

   ```makefile
   SUBDIRS = src
   ```

   **src/Makefile.am**

   ```makefile
   bin_PROGRAMS = hello
   hello_SOURCES = hello.c
   ```

3. 执行autoscan生成configure.scan文件，把configure.scan重命名为configure.ac

   ```sh
   $ autoscan
   $ mv configure.scan configure.ac
   ```

   编辑configure.ac，在AC_INIT后增加AM_INIT_AUTOMAKE

   ```makefile
   #                                               -*- Autoconf -*-
   # Process this file with autoconf to produce a configure script.
   
   AC_PREREQ([2.69])
   AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
   AM_INIT_AUTOMAKE
   AC_CONFIG_SRCDIR([src/hello.c])
   AC_CONFIG_HEADERS([config.h])
   
   # Checks for programs.
   AC_PROG_CC
   
   # Checks for libraries.
   
   # Checks for header files.
   
   # Checks for typedefs, structures, and compiler characteristics.
   
   # Checks for library functions.
   
   AC_CONFIG_FILES([Makefile
                    src/Makefile])
   AC_OUTPUT
   ```

   此时文件结构如下：

   ```sh
   $ tree
   .
   ├── autoscan.log
   ├── configure.ac
   ├── Makefile.am
   └── src
       ├── hello.c
       └── Makefile.am
   
   1 directory, 5 files
   ```

4. 执行aclocal生成aclocal.m4

   ```sh
   $ aclocal
   $ tree
   .
   ├── aclocal.m4
   ├── autom4te.cache
   │   ├── output.0
   │   ├── requests
   │   └── traces.0
   ├── autoscan.log
   ├── configure.ac
   ├── Makefile.am
   └── src
       ├── hello.c
       └── Makefile.am
   
   2 directories, 9 files
   ```

5. 执行autoconf生成configure文件

   ```sh
   $ autoconf
   $ tree
   .
   ├── aclocal.m4
   ├── autom4te.cache
   │   ├── output.0
   │   ├── output.1
   │   ├── requests
   │   ├── traces.0
   │   └── traces.1
   ├── autoscan.log
   ├── configure
   ├── configure.ac
   ├── Makefile.am
   └── src
       ├── hello.c
       └── Makefile.am
   
   2 directories, 12 files
   ```

6. 执行autoheader生成config.h.in文件

   ```sh
   $ autoheader
   $ tree
   .
   ├── aclocal.m4
   ├── autom4te.cache
   │   ├── output.0
   │   ├── output.1
   │   ├── requests
   │   ├── traces.0
   │   └── traces.1
   ├── autoscan.log
   ├── config.h.in
   ├── configure
   ├── configure.ac
   ├── Makefile.am
   └── src
       ├── hello.c
       └── Makefile.am
   
   2 directories, 13 files
   ```

7. 创建必要的文档文件：NEWS README AUTHORS ChangeLog，文件可以没有内容

   ```sh
   $ touch NEWS README AUTHORS ChangeLog
   ```

8. 执行automake --add-missing根据Makefile.am生成Makefile.in文件以及生成compile install-sh missing INSTALL COPYING depcomp等文件，这几个文件都是automake工具集里的软链接

   ```sh
   $ automake --add-missing
   $ tree
   .
   ├── aclocal.m4
   ├── AUTHORS
   ├── autom4te.cache
   │   ├── output.0
   │   ├── output.1
   │   ├── requests
   │   ├── traces.0
   │   └── traces.1
   ├── autoscan.log
   ├── ChangeLog
   ├── compile -> /usr/share/automake-1.14/compile
   ├── config.h.in
   ├── configure
   ├── configure.ac
   ├── COPYING -> /usr/share/automake-1.14/COPYING
   ├── depcomp -> /usr/share/automake-1.14/depcomp
   ├── INSTALL -> /usr/share/automake-1.14/INSTALL
   ├── install-sh -> /usr/share/automake-1.14/install-sh
   ├── Makefile.am
   ├── Makefile.in
   ├── missing -> /usr/share/automake-1.14/missing
   ├── NEWS
   ├── README
   └── src
       ├── hello.c
       ├── Makefile.am
       └── Makefile.in
   
   2 directories, 25 files
   ```

9. 开始编译

   ```sh
   $ ./configure
   $ make
   $ make install
   ```

## autotools介绍

autotools是一系列工具，上面用到的命令都来自于autotools

- autoscan

  可以通过调用autoscan命令得到一个初始化的configure.scan文件，然后重命名为configure.ac后在此基础上编辑configure.ac

  autoscan会扫描源码，并生成一些通用的宏调用，输入的声明以及输出的声明

- aclocal

  configure.ac实际上是依靠宏展开来得到configure；因此，能否成功生成取决于宏定义是否能够找到

  autoconf会从自身安装路径下寻找事先定义好的宏，然而对于像automake、libtool、等第三方扩展宏autoconf便无从知晓；因此aclocal将在configure.ac同一个目录下生成aclocal.m4，在扫描configure.ac的过程中将第三方扩展和开发者自己编写的宏定义复制进去；如此一来，autoconf遇到不认识的宏时就会从aclocal.m4中查找

- autoconf

  autoconf是用来生成configure文件的；configure文件是一个脚本，它能设置源程序来适应各种不同的操作系统平台，并根据不同的系统来生成合适的Makefile，从而使得源码能在不同的操作系统平台上被编译出来

- autoheader

  autoheader命令扫描configure.ac文件，并确定如何生成config.h.in；每当configure.ac变化时，都可以通过执行autoheader更新config.h.in

  在configure.ac中通过AC_CONFIG_HEADERS([config.h])告诉autoheader应当生成config.h.in的路径

  config.h包含了大量的宏定义，其中包括软件包的名字等信息，程序可以直接使用这些宏；更重要的是，程序可以根据其中对目标平台的可移植相关的宏，通过条件编译动态的调整编译行为

- automake

  automake命令根据Makefile.am文件生成Makefile.in文件

## configure.ac常用宏

|        宏         |                             功能                             |
| :---------------: | :----------------------------------------------------------: |
|     AC_PREREQ     |                   声明autoconf要求的版本号                   |
|      AC_INIT      |                定义软件名称、版本号、联系方式                |
| AM_INIT_AUTOMAKE  |                         指定编译参数                         |
| AC_CONFIG_SRCDIR  |   用来检查所指定的源码文件是否存在，来确定源码目录的有效性   |
| AC_CONFIG_HEADER  | 指定生成的配置文件名称（一般是config.h），用于生成config.h文件，以便autoheader命令使用 |
|    AC_PROG_CC     |               用于检查当前系统C编译器是否存在                |
|  AC_PROG_RANLIB   |                        用于生成静态库                        |
|  AC_PROG_LIBTOOL  |                        用于生成动态库                        |
|    AM_PROG_AR     |         生成静态库时使用，用于指定打包工具，一般是ar         |
|  AC_CONFIG_FILES  |         告知autoconf本工程生成哪些相应的Makefile文件         |
|     AC_OUTPUT     |        一个必须的宏，用于最后，用于输出需要生成的文件        |
|    AC_PROG_CXX    |              用于检查当前系统C++编译器是否存在               |
|   AC_CHECK_LIB    |                 检查库文件以及库文件中的方法                 |
| PKG_CHECK_MODULES |                利用pkg-config生成_CFLAGS_LIBS                |
|     AC_SUBST      |               输出能够被Makefile.am使用的变量                |
|      LT_INIT      |                         检查libtool                          |
|   AC_MSG_ERROR    |                      打印错误信息并退出                      |
|    AC_MSG_WARN    |                      打印警告信息不退出                      |

## Makefile.am常用规则

- 编译类型

  |  编译类型   |     说明      |    使用方式     |
  | :---------: | :-----------: | :-------------: |
  |  PROGRAMS   |  可执行程序   |  bin_PROGRAMS   |
  |  LIBRARIES  |    库文件     |  lib_LIBRARIES  |
  | LTLIBRARIES | libtool库文件 | lib_LTLIBRARIES |
  |   HEADERS   |    头文件     | include_HEADERS |
  |   SCRIPTS   |   脚本文件    | script_SCRIPTS  |
  |    DATA     |   数据文件    |    conf_DATA    |

- 安装目录

  |      安装路径      |     使用方式     |
  | :----------------: | :--------------: |
  | ${exec_prefix}/bin |   bin_编译类型   |
  | ${exec_prefix}/lib |   lib_编译类型   |
  | ${prefix}/include  | include_编译类型 |
  |       不安装       | noinst_编译类型  |

  prefix值通过执行./configure期间使用参数--prefix=指定，默认为/usr/local

  exec_prefix值为${prefix}

  script_SCRIPTS和conf_DATA中的安装目录需要手动配置

- 编译目标

  |     参数      |               含义               |
  | :-----------: | :------------------------------: |
  |   _SOURCES    |            源代码文件            |
  |    _LIBADD    |     目标为lib时需要链接的库      |
  |    _LDADD     |     目标为bin时需要链接的库      |
  |   _LDFLAGS    | 对于-L, -l, -shared, -fpic等选项 |
  | _LIBTOOLFLAGS |       libtool编译时的选项        |

## libtool

libtool是GNU autotools工具集中的一个，用于方便编译和安装动态库；通常结合autoconf和automake一起使用

使用libtool编译动态库模板：

**Makefile.am**

```makefile
# Build a libtool library, libhello.la for installation in libdir.
lib_LTLIBRARIES = libhello.la
libhello_la_SOURCES = hello.c foo.c
libhello_la_LDFLAGS = -version-info 3:12:1
```

## pkg-config

pkg-config是linux中用于管理库的依赖关系以及提供库的编译和链接参数的一个工具，其原理通过把库的依赖关系和编译链接参数记录在.pc文件中，然后可以通过pkg-config命令获取这些信息；pkg-config不属于GNU autotools工具集，但可以结合autoconf和automake一起使用

- 编译库时使用autoconf automake生成和安装与库对应的.pc文件

  在configure.ac文件中AC_CONFIG_FILES 宏中增加xxx.pc以及在根目录创建xxx.pc.in文件

  ```makefile
  AC_CONFIG_FILES([Makefile
                   src/Makefile
                   xxx.pc])
  ```
  配置AC_CONFIG_FILES后执行configure脚本时会在根目录输出xxx.pc文件
  
  **xxx.pc**
  
  ```makefile
  prefix=@prefix@
  exec_prefix=@exec_prefix@
  libdir=@libdir@
  includedir=@includedir@
  
  Name: xxx
  Description: demo for use autotools and pkg-config
  Version: @PACKAGE_VERSION@
  Libs: -L${libdir} -lxxx
  Cflags: -I${includedir}/xxx
  ```
  
  要使configure脚本输出xxx.pc文件就需要xxx.pc.in文件作为模板，configure脚本会将目标文件中双@号标识的变量替换为其实际的值
  
  要在执行`make install`的时候把.pc文件安装到目标目录中还需要修改Makefile.am文件
  
  **Makefile.am**
  
  ```makefile
  pkgconfigdir = $(libdir)/pkgconfig
  pkgconfig_DATA = say.pc
  ```

- 编译bin文件时通过pkg-config链接对应的库

  在configure.ac文件中增加

  ```makefile
  PKG_CHECK_MODULES(xxx, xxx >= x.x)
  ```

  表示在执行configure脚本的时候会去检查符合版本的xxx库是否存在，如果存在会把对应库的编译参数和链接参数放到变量**xxx_CFLAGS**和**xxx_LIBS**中

  然后在Makefile.am文件中声明bin目标时使用这两个变量

  ```makefile
  bin_PROGRAMS = hello
  hello_SOURCES = hello.c
  AM_CFLAGS = $(xxx_CFLAGS)
  hello_LDADD = $(xxx_LIBS)
  ```

## 创建动态库构建工程（使用libtool和pkg-config）

1. 准备源码文件如下：

   ```shell
   .
   ├── Makefile.am
   ├── say.pc.in
   └── src
       ├── Makefile.am
       ├── say_hello.c
       └── say_hello.h
   
   1 directory, 5 files
   ```

   say_hello.c文件实现一个接口，say_hello.h是对外头文件声明了这个接口

   **Makefile.am**

   ```makefile
   SUBDIRS = src
   
   pkgconfigdir = $(libdir)/pkgconfig
   pkgconfig_DATA = say.pc
   ```

   通过pkgconfigdir和pkgconfig_DATA声明.pc文件的安装目录和需要安装的.pc文件

   **say.pc.in**

   ```makefile
   prefix=@prefix@
   exec_prefix=@exec_prefix@
   libdir=@libdir@
   includedir=@includedir@
   
   Name: say
   Description: demo for use autotools and pkg-config
   Version: @PACKAGE_VERSION@
   Libs: -L${libdir} -lsay
   Cflags: -I${includedir}/say
   ```

   用于创建say.pc文件的模板文件，文件中用双@号标识的变量最终会被替换为变量的值

   **src/Makefile.am**

   ```makefile
   lib_LTLIBRARIES = libsay.la
   libsay_la_SOURCES = say_hello.c
   libsay_la_LDFLAGS = -version-info @LT_CURRENT@:@LT_REVISION@:@LT_AGE@
   
   sayincludedir = $(includedir)/say
   sayinclude_HEADERS = say_hello.h
   ```

   `lib_LTLIBRARIES = libsay.la`声明libtool动态库构建目标

   `libsay_la_LDFLAGS = -version-info @LT_CURRENT@:@LT_REVISION@:@LT_AGE@`将库的版本号信息加入到链接参数中，其中LT_CURRENT等3个变量需要在configure.ac文件中定义

2. 执行autoscan生成configure.scan文件，把configure.scan重命名为configure.ac并编辑

   ```makefile
   #                                               -*- Autoconf -*-
   # Process this file with autoconf to produce a configure script.
   
   AC_PREREQ([2.69])
   AC_INIT([say], [1.0.0], [BUG-REPORT-ADDRESS])	# 1. 修改软件名称和软件版本
   LT_INIT																				# 2. 初始化libtool
   AM_INIT_AUTOMAKE															# 3. 初始化automake
   AC_CONFIG_SRCDIR([src/say_hello.c])
   AC_CONFIG_HEADERS([config.h])
   
   # Checks for programs.
   AC_PROG_CC
   
   # Checks for libraries.
   
   # Checks for header files.
   
   # Checks for typedefs, structures, and compiler characteristics.
   
   # Checks for library functions.
   
   AC_SUBST([LT_CURRENT], [1])										# 4. 定义so库版本号
   AC_SUBST([LT_REVISION], [0])
   AC_SUBST([LT_AGE], [0])
   
   AC_CONFIG_FILES([Makefile
                    src/Makefile
                    say.pc])											# 5. 添加.pc文件为configure脚本输出文件
   AC_OUTPUT
   ```

3. 执行libtoolize命令生成ltmain.sh

   ```shell
   $ libtoolize
   ```

4. 执行aclocal、autoconfig、autoheader、automake等步骤同创建autoconf&automake工程步骤一致

5. 开始编译

   ```shell
   $ ./configure 
   $ make
   $ make install
   ```

## 创建依赖第三方库的bin构建工程

1. 准备源码文件如下：

   ```shell
   .
   ├── Makefile.am
   └── src
       ├── hello.c
       └── Makefile.am
   
   1 directory, 3 files
   ```

   **Makefile.am**

   ```makefile
   SUBDIRS = src
   ```

   **src/hello.c**

   ```c
   #include "say_hello.h"
   
   int main(int argc, char **argv)
   {
     say_hello();
     return 0;
   }
   ```

   **src/Makefile.am**

   ```makefile
   bin_PROGRAMS = hello
   hello_SOURCES = hello.c
   AM_CFLAGS = $(say_CFLAGS)
   hello_LDADD = $(say_LIBS)
   ```

2. 执行autoscan生成configure.scan文件，把configure.scan重命名为configure.ac并编辑

   ```makefile
   #                                               -*- Autoconf -*-
   # Process this file with autoconf to produce a configure script.
   
   AC_PREREQ([2.69])
   AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
   LT_INIT																					# 1. 初始化libtool
   AM_INIT_AUTOMAKE																# 2. 初始化automake
   AC_CONFIG_SRCDIR([src/hello.c])
   AC_CONFIG_HEADERS([config.h])
   
   # Checks for programs.
   AC_PROG_CC
   
   # Checks for libraries.
   PKG_CHECK_MODULES(say, say >= 1.0)							# 3. 检查依赖的库say
   
   # Checks for header files.
   
   # Checks for typedefs, structures, and compiler characteristics.
   
   # Checks for library functions.
   
   AC_CONFIG_FILES([Makefile
                    src/Makefile])
   AC_OUTPUT
   ```

   这里第一步初始化libtool如果不添加也可以编译成功，区别是添加这个之后在链接程序的时候会使用libtool工具，libtool工具会把库的安装路径添加到gcc的链接参数中作为rpath写入bin文件中，这样即使say库没有安装在标准lib目录也能在执行bin文件的时候根据rpath成功加载say库并成功执行程序

   可以通过readelf命令查看bin文件中的rpath值

   ```shell
   readelf -d hello | grep RPATH
   ```

3. 执行libtoolize命令生成ltmain.sh

4. 执行aclocal、autoconfig、autoheader、automake等步骤同创建autoconf&automake工程步骤一致

5. 开始编译

   ```shell
   $ export PKG_CONFIG_PATH=/path/to/pkgconfig
   $ ./configure
   $ make
   $ make install
   ```

   [^注意]: 这里编译步骤多一步创建环境变量PKG_CONFIG_PATH的步骤，是因为执行./configure的时候会根据这个路径遍历.pc文件去查找依赖的say库
## 官方文档地址

[automake]: https://www.gnu.org/software/automake/manual/automake.html
[Autoconf]: https://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.71/autoconf.html



