# Verilog-to-Bitstream2.0
VTB的依赖工具、搭建步骤、环境配置、遇到的问题及解决办法

## VTB依赖的工具
* VTR 7.0 (下载地址：https://github.com/verilog-to-routing/vtr-verilog-to-routing/releases/tag/vtr_v7)
* vtr-to-bitstream_v2.1.patch (下载地址：https://github.com/eddiehung/eddiehung.github.io/releases/tag/vtb_v2.1) 
* Torc-1.0 (下载地址：http://svn.code.sf.net/p/torc-isi/code/tags/torc-1.0)  #由于torc比较大，所以没有上传文件。
* Yosys-0.9 (下载地址：http://www.clifford.at/yosys/download.html)
* abc (下载地址：https://github.com/berkeley-abc/abc)
* Xilinx ISE 14.7 for Linux (官网下载:https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive-ise.html) (文件中附有证书）

## 搭建步骤：
* 1. 下载`VTR 7.0`，在进行后续步骤之前先进行编译，编译通过后再进行下一步。（即在VTR7.0目录下执行`Makefile`文件， make 时出现的错误后面会详细介绍，如果下载的本目录下的VTR可忽略1,2步）
* 2. 下载补丁`vtr-to-bitstream_v2.1.patch`，放在VTR7.0目录下（指VTR7.0文件夹下，后同），执行命令：
```Bash
                patch -p1 < vtr-to-bitstream_v2.1.patch
```
* 3. 将`torc-1.0`和`yosys-yosys-0.9`放入VTR7.0目录下，分别重命名为`torc`和`yosys`
* 4. 将`abc`文件放入yosys目录下，并将yosys目录下Makefile文件中`ABCREV = 3709744`改为`ABCREV = default`。
* 5. 配置运行环境（见下文）
* 6. 下载Xilinx ISE 14.7，必须要full license（安装时注意环境：ubuntu-32位，安装教程可自行百度）
* 7. 配置环境变量，我们需要的只是ISE的xdl2ncd和bitgen等模块，且需要在命令行直接调用这些模块，因此需要配置环境变量：
```Bash
        export PATH=$PATH:pathTo/Xilinx/14.7/ISE_DS/ISE/bin/lin  #pathTo由ISE的下载路径决定
```        
* 8. 在VTR7.0目录下执行重新执行`Makefile`文件，若运行结果如下图，则说明编译成功。<br>
![image](https://github.com/meijianyue/Verilog-to-Bitstream2.0/blob/master/Result.png)

## 运行环境配置
* VTR7.0编译时依赖一系列的包，torc工具编译时需要gcc、Boost和Subversion，yosys编译时需要clang和git，如下所示：<br>
        -- ubuntu-18.04(64位)  <br>
        -- gcc 4.8.5 && g++ 4.8.5(安装教程：https://www.jianshu.com/p/f66eed3a3a25)  <br>
        -- clang version 3.9.1 (安装:https://blog.csdn.net/DumpDoctorWang/article/details/84567757) <br>
        -- Subversion 1.9.7  <br>
        -- boost-1.54.0 (安装教程：https://blog.csdn.net/this_capslock/article/details/47170313) <br>
        -- git 2.7.4  <br>
         * 注意：如果clang或者其他依赖包的版本与以上给出的版本不同，可能会报许多奇怪的错误。可以慢慢找解决方案，最好还是保持版本一致。 <br>
     
* 参考VTR8.0的手册，VTR运行需要配置如下包：
（注意：VTR7.0的编译和运行其实只需要下述包中的几个，大部分可不必下载，如果需要节省内存，可以先不下载下述包，直接执行编译，然后根据报错时出现的提示下载所缺的包。）
```Bash
    sudo apt-get install \
      build-essential \
      flex \
      bison \
      cmake \
      fontconfig \
      libcairo2-dev \
      libfontconfig1-dev \
      libx11-dev \
      libxft-dev \
      libgtk-3-dev \
      perl \
      liblist-moreutils-perl \
      python 
```
* 用于文件生成的包：
```Bash
    sudo apt-get install \
      doxygen \
      python-sphinx \
      python-sphinx-rtd-theme \
      python-recommonmark
```
* 开发依赖包：
```Bash
    sudo apt-get install \
      git \
      valgrind \
      gdb \
      ctags
```

## 编译遇到的问题及解决办法
### VTR7.0预编译
* 前面搭建步骤中提到过，下载VTR7.0后需要预先编译，以预先解决部分编译问题。
* 打开终端，进入`VTR7.0`根目录，执行make <br>
* 可能会出现的错误： <br>
* 编译odin时报错：对`ODIN/SRC/simulate_blif.c`文件中一些函数“未定义的引用”
    这些函数定义时前面都有"inline"关键字，在gcc编译过程中，该关键字最好不要出现在.c文件的函数定义中，原因自行百度。因此，只需在该.c文件中找到所有报错的函数的定义，去掉`inlin`关键字即可。 
    
* 编译abc时报错：“src/misc/espresso/unate.c”文件中关于"restrict"的error  <br>
    gcc 5.4.0版本编译时，`restrict`被识别为关键字，不能用作变量或者函数参数名。将所有`restrict`改为`_restrict`即可。  <br>

* 编译ace2时报错：文件`ace2/ace.c`里的函数`print_status`中，对`Ace_ObjInfo`未定义的引用  <br>
    还是`inline`关键字的问题，将函数`Ace_ObjInfo`定义前面的`inline`去掉即可。  <br>

* 注意事项：vpr工具提供图形化界面，如果在编译后想要执行vtr流程并查看图形化界面，可修改 vpr/Makefile 文件中的参数：  <br>
   ` ENABLE_GRAPHICS = true`  <br>
    
### VTB编译
 配置好VTB的依赖包和运行环境后，在 “VTR7.0” 根目录下执行 make，完成整个Verilog-to-Bitstream编译。以下给出编译过程中可能出现的错误以及解决方法。
 打开终端，进入`VTR7.0`根目录，执行`make` <br>
 可能出现的错误：  <br>
* 编译torc时报错： <br>
    1. 如果在执行“cd torc && svn cleanup && svn up”时报错："client version is old ..."（类似这样），是由于
Subversion版本过低，需要更新至更高版本。  <br>
    2. 如果在执行“cd torc && svn cleanup && svn up”时报错："...不是副本目录"，可采取如下做法： <br>
    （1）打开torc文件，按找到隐藏文件.svn(ctrl+h可查看隐藏文件)，把他删除  <br>
    （2）在“VTR7.0”根目录下(torc文件所在目录)执行命令：svn checkout http://svn.code.sf.net/p/torc-isi/code/tags/torc-1.0 torc   <br>
    （3）此时会生成新的.svn文件，下载最新torc需要较长时间（因为我们之前下载的torc已是最新版本，.svn文件生成后可以直接中断下载(Ctrl+c))  <br>
    （4）再次在“VTR7.0”根目录执行 make ，svn升级torc时会产生冲突，直接选择“(r)mark resolved”即可。原因是svn的锁机制，前面中断下载后，小部分中断前正在下载的文件夹会被锁住，我们并未找到它们并执行"svn clean up"，但是这不影响后续编译和运行  <br>
    
* 编译yosys时会报错：  <br>
    1. 关于`tcl.h`的报错，需要下载`tcl8.6-dev`包  <br>
    2. 关于`readline.h`的报错，需要下载`libreadline-dev`包  <br>
    3. 有些.hpp文件报错：‘uint32_t/uint8_t/uint16_t’ has not been declared，在相应头文件里加上`#include <inttypes.h>`  <br>
    4. `Flattening.cpp`报错：关于函数重定义，默认参数的错误，原因是c++中，在声明和定义含默认参数的函数时，声明、定义中只有一个能包含默认参数：  <br> 
```cpp
        //错误写法
        int add(int a, int b=10);  //函数声明   
        int add(int a, int b=10) {...} //函数定义  
        //正确写法
        int add(int a, int b);  //函数声明   
        int add(int a, int b=10) {...} //函数定义 
```      
* 报错的函数的声明和定义中都有默认参数，找到其在Flattening.hpp头文件中的函数声明，去掉 ` =默认参数值`即可  <br>
    5. 报错：`torc/src/torc/generic/edif/Decompiler.cpp:37:13: error: ‘std::__cxx11::string {anonymous}::trimLeading(const string&)’ defined but not used [-Werror=unused-function]` , 删除文件`/torc/src/torc/Makefile.targets`中第59行: `-Werror \`即可。这里只是某个函数定义了却未使用的警告，因为有`-Werror`，所有警告被当作错误，故去掉即可。  <br>
    
* 提示`.o`文件中函数出现未定义引用，是由于之前编译过程中产生的`.o`和`.d`文件在后面进行调用时出现错误导致的。哪个文件目录下报错，就在哪个目录下执行下列命令删除`.o`和`.d`文件，重新执行编译。
```Bash 
          find . -name "*.o" | xargs rm -rfv

          find . -name "*.d" | xargs rm -rfv
```
* 工具代码本身并没有错误，如果在编译时出现代码错误的提示，先尝试把对应文件下编译生成的`.o`&`.d`文件删除，如果还是不行就重新安装`boost`或者更改`gcc`和`g++`和`clang`版本，与上面给出的版本保持一致。


* 编译Xilinx ISE提供的bit流生成模块时报错  <br>
    1. 第一种错误是找不到`partgen`的路径，这就是前面搭建步骤中最后一步环境变量有问题，需要正确配置  <br>
    2. 第二种错误：Can't locate File/Which.pm in @INC (you may need to install the File::Which module) ，执行命令`cpan File::Which`安装即可  <br>


* 编译完成后，可以测试一下VTB能否从`.v`文件顺利生成`.bit`文件，新建一个`test`文件夹，在终端进入该文件夹后执行命令：  <br>
```Bash
    $VTR_ROOT/vtr_flow/scripts/run_vtr_flow.pl $VTR_ROOT/vtr_flow/benchmarks/verilog/mkPktMerge.v \                
    $VTR_ROOT/vtr_flow/arch/xilinx/xc6vlx240tff1156.xml
    # $VTR_ROOT表示"VTR7.0"根目录
```

## 分步执行VPR以及显示布局布线界面
 后面我们通过修改`run_vtr_flow.pl`文件，使`VPR`可以分步运行。
 该脚本用于调用执行VTB，可通过设置参数控制执行阶段：
```Bash
   -starting_stage odin -ending_stage bitstream  //参数设定
```
 初始阶段划分： `odin  abc  ace  prevpr  vpr  bitstream` <br>
 修改后： `odin  abc  ace  prevpr "pack  place  route (vpr)" bitstream`<br>
 <br>
 方法：
* 新增加了一个参数`"-vpr_stage   XXX"`如果命令行有该参数，则区分pack、place和route；如果命令行没有该参数，直接执行整个vpr流程。
#### 显示布局布线的界面
  删除`run_vtr_flow.pl`代码中的`--nodisp`参数，此参数指不展示界面，将他删除之后运行脚本，就会在布局布线阶段显示界面。<br>
  <br>
  除了删除参数外，在vpr/SRC/base文件夹下有个`draw.c`文件，其中有几个断言函数会阻止界面运行，位于代码的第528、548、1226、1300、1960、1966行，原因尚不得知。将其全部注释，然后重新编译vpr，界面即可正常运行。

