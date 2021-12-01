# Xenomai

## 基本介绍

Xenomai是一个双内核实时操作系统。一般是结合Linux内核和另一个实时内核，以期既能实现实时操作系统，又能完成对Linux操作系统的资源复用。

1. 基本框架
    - ![基本框架](https://raw.githubusercontent.com/Richardhongyu/pic/main/20211201183839.png)
    - 组成结构
      - RTDM写驱动相关
      - ipipe作为虚拟化层，主要是对中断虚拟化
        - 这一层是为了帮助linux跑在另一个系统上的
      - Cobalt(XEnomai Nucleus)是一个实时内核，和Linux运行在同一层级
        - 这一层是直接管理硬件资源，需要跑在hardware上，就像一个小的操作系统
      - Skins是一个和上层api进行沟通的组件，通过这个组件实现多种接口，POSIX，Vxwork，以实现用户态的实时性

## Xenomai的安装和启动

1. 如何利用qemu启动Xenomai，并进行开发
   - [PPT第二小节](./Xenomai/xenomai-1.pptx)
2. 官网安装教程
   - [链接](https://source.denx.de/Xenomai/xenomai/-/wikis/Setting_Up)

## 源码解析

Xenomai主体是两个部分，ipipe和实时内核cobalt。ipipe是以补丁形式提供，直接修改了内核的代码，作用是根据优先级将中断分发给cobalt内核和Linux内核。实时内核cobalt则作为module插入到linux系统中，一起启动。

Xenomai还有两个小的组件，负责提供生态相关的内容。RTDM是驱动相关，各类Skin负责模拟编程接口，如vxworks。

下面从各个模块具体介绍：

### ipipe

1. ipipe是如何启动的
   - [PPT第一小节](./Xenomai/xenomai-2.pptx)

### cobalt

1. cobalt是如何启动的
   - [PPT第一小节](./Xenomai/xenomai-2.pptx)
2. 调度子系统
   - [PPT第一小节](./Xenomai/xenomai-2.pptx)
3. 中断子系统
4. 时间子系统
5. 内存管理子系统
6. IO子系统
7. 虚拟文件子系统
8. 锁相关机制

### RTDM

### skins

## 相关资料

### Xenomai官方资料

1. Xenomai的[官网](https://source.denx.de/Xenomai/xenomai/-/wikis/home)
2. Xenomai的[文档](https://xenomai.org/documentation/xenomai-3/html/xeno3prm/index.html)，里面关于ipipe的内容比较少
3. ipipe的[文档](https://source.denx.de/Xenomai/ipipe-x86/-/blob/ipipe-x86-5.4.y/Documentation/ipipe.rst)
4. Xenomai的[发展历史](https://source.denx.de/Xenomai/xenomai/-/wikis/History)
5. Xenomai的宣传视频，介绍了一些基本的情况：[视频](https://www.youtube.com/watch?v=Le1O2Hy8KUk)

### Xenomai解读博客

1. Xenomai的系列中文解析文章，应该是目前最全的（英文世界中解读源码的博客没有找到）：[文章](https://www.cnblogs.com/wsg1100/p/13836497.html)

### Xenomai课程

1. 这门[课程](http://dit.unitn.it/~abeni/RTOS/)用到了Xenomai，里面有大量的Xenomai相关资料和相关代码

### 在让Linux具有实时性上的双内核和单内核之争

1. 在让Linux具有实时性上的双内核和单内核之争
   - [PPT第一小节](./Xenomai/xenomai-1.pptx)
2. RTL组织单内核路线的一个演讲，给出了单双内核的latency测试结果：[视频](https://www.youtube.com/watch?v=BKkX9WASfpI)

### 实时/嵌入式操作系统学术相关

1. 实时/嵌入式学术会议和关注内容
   - [PPT第三小节](./Xenomai/xenomai-1.pptx)
   - 嵌入式和实时操作系统
     - 嵌入式操作系统的特点是可以运行在嵌入式设备上，进而控制其它外设；实时操作系统的特点是任务时间可预测；
     - 实时操作系统一般运行在工业设备上，一般是嵌入式的；
     - 嵌入式操作系统对时延的要求不一定特别高，所以不一定是实时的；
   - 顶级会议
     - 实时操作系统
       - RTSS CCFA *
       - RTAS CCFB
       - ECRTS CCFB
       - RTCSA CCFC
       - RTNS CCFC
     - 嵌入式系统
       - EMSOFT | ES Week (A; CCF B) *
       - CODES+ISSS | ES Week (A; CCF B)
       - Ada-Europe (B)
       - FPGA (B; CCF B)
       - NOCS (B; CCF C)
   - 顶级实验室
     - https://www4.cs.fau.de/Research/Sloth/
     - https://www.sra.uni-hannover.de/Research/
   - 可研究方向
     - 嵌入式操作系统
     - 实时操作系统
       - MERT - a multi-environment real-time operating system(SOSP,1975)
       - Thoth, a portable real-time operating system (Extended Abstract)(SOSP,1977)
       - EMERALDS: a small-memory real-time microkernel (SOSP,1999)
     - 形式化验证
       - Automatic Verification of Application-Tailored OSEK Kernels(FMCAD,2017)
     - 安全
       - RapidPatch: Firmware Hotpatching for Real-Time Embedded Devices（USENIX Security ，2022）
   - 参考资料
     - https://zhuanlan.zhihu.com/p/61362833
     - https://github.com/automaticdai/realtime-embedded-conferences
     - https://site.ieee.org/tcrts/education/seminal-papers/
2. 实时/嵌入式操作系统实例
   - 实时操作系统
     - 普通的实时操作系统
     - 让Linux成为实时操作系统
       - [A linux-based real-time operating system. (Barabanov, Michael论文，1997).](./RT-Linux/BarabanovThesis.pdf)
         - 第一个可以兼容Linux的实时操作系统（RT-Linux）,采用双内核路线。
       - [Xenomai-Implementing a RTOS emulation framework on GNU/Linux（White Paper, 2004）](https://source.denx.de/Xenomai/xenomai/-/wikis/White_Paper)
         - Xenomai的白皮书
   - 嵌入式操作系统
     - The Tock Embedded Operating System（Sensys,2017 ）
     - [Sloth: A Minimal-Effort Kernel for Embedded Systems(SOSP,2009)](https://www4.cs.fau.de/Research/Sloth/)
       - 这个项目的后续工作发了RTSS RTAS，均是嵌入式系统的顶会
     - dOSEK: The Design and Implementation of a Dependability-Oriented Static Embedded Kernel（RTAS, 2015, best paper）
3. 实时/嵌入式操作系统的组件发展
   - 双内核路线下的Linux实时操作系统
     - 硬件虚拟层
       - 硬件虚拟层是双内核路线中的关键一环，RTHAL是RTLinux的论文提出来的，被RTAI仿照实现，RTLinux的项目组后来申请了专利。所以后来社区采用了ADEOS。
       - RTHAL是在RT-Linux那个论文中提出的，没有论文
       - [Adaptive domain environment for operating systems.（Opersys inc, 2001）](./Xenomai/adeos.pdf)
         - ADEOS硬件虚拟层，采用不同的技术路线替换了RT-Linux提出的RTHAL，规避了专利问题。
         - [Wiki](https://en.wikipedia.org/wiki/Adaptive_Domain_Environment_for_Operating_Systems)
         - [主页1](https://www.opersys.com/adeos/)
         - [主页2](https://gna.org/projects/adeos/ )
           - 这个主页是开源项目活跃时的协作网址。现在这个项目和xenomai合并了，因为Philippe Gerum是Xenomai和ADEOS项目的创始人。
       - [Life with adoes.(Philippe Gerum, 2005)](./Xenomai/life-with-adeos.pdf)
         - ADEOS的实现细节
       - [Study and Comparison of the RTHAL-based and ADEOS-based RTAI Real-time Solutions for Linux. (IEEE IMSCCS.2006)](./Xenomai/Study_and_Comparison_of_the_RTHAL-Based_and_ADEOS-Based_RTAI_Real-time_Solutions_for_Linux.pdf)
         - 比较ADEOS和RTHAL两种硬件虚拟层的技术
     - 驱动
        - [The Real-Time Driver Model and First Applications. (Real-Time Linux Workshop, Lille, France. 2005.)](./Xenomai/RTDM-and-Applications.pdf)
4. 实时/嵌入式操作系统比较综述
   - The Real-Time Linux Kernel: A Survey on PREEMPT_RT(ACM Computing Surveys. 2019)
     - 这个综述中比较了RTL和其它的双内核实时操作系统
   - [Performance evaluation of Xenomai 3.(Proceedings of the 17th Real-Time Linux Workshop (RTLWS),2015.)](http://wiki.csie.ncku.edu.tw/embedded/xenomai/rtlws_paper.pdf)
     - 评测xenomai2，3和PREEMPT_RT补丁的实时情况
     - 这个项目组还在youtube上放出了一个小项目的视频以及对应的开发文档
   - [Real-time operating systems（Real-Time Systems， 2004）](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.124.7500&rep=rep1&type=pdf)
5. 实时/嵌入式操作系统的应用
   - [Real-Time Linux Testbench on Raspberry Pi 3 using Xenomai.（Master paper）](./Xenomai/FULLTEXT01.pdf)
     - 树莓派上评测Xenomai性能
6. 其它学习资料
   - [IEEE官方指导](https://site.ieee.org/tcrts/)