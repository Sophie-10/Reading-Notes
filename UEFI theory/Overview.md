UEFI  是BIOS 的替代者

# 1、BIOS
BIOS 基本输入/输出系统
### 1.1、BIOS 在计算机系统中的作用
```
1、加电自检程序
2、系统初始化代码，包括硬件设备的初始化、创建BIOS中断向量等
3、基本的外围IO处理的子程序代码
4、CMOS 设置程序
```

### 1.2、BIOS 的缺点
```
1、开发效率低
2、性能差
3、功能扩展性差，升级慢
4、安全性
5、不支持从硬盘2T以上的地址引导
```

# 2、UEFI
UEFI (Unified Extensible Firmware Interface，统一可扩展固件接口)
定义了操作系统和平台固件直接的接口

```
UEFI 实现一般分为两部分：
1、平台初始化
2、固件 -- 操作系统接口
```

### 2.1 UEFI 组成
```
启动服务（Boot Services，BS）
运行时服务（Runtime Services，RT）
隐藏在BS之后的丰富的Protocol
```
![image](https://github.com/Sophie-10/Reading-Notes/blob/master/images/UEFI/1-1EFI%E7%B3%BB%E7%BB%9F%E7%BB%84%E6%88%90.png?raw=true)
图1-1


```
从操作系统加载器（OS Loader）被加载，到 OS Loader 执行 ExitBootServices（）的这段时间，是从UEFI环境向操作系统过度的过程。在这个过程中，OS Loader 可以通过 BS 和 RT 使用 UEFI 提供的服务，将计算机系统资源逐渐转移到自己手中，这个过程称为 TSL(Transient System Load)。
当 OS Loader 完全掌握了计算机系统资源时，BS 也就完成了它的使命。OS Loader 调用 ExitBootServices（）结束BS并回收BS 占用的资源，之后计算机系统进入UEFI Runtime阶段。
在Runtime 阶段只有运行时服务继续为OS 提供服务，BS已经从计算机系统中销毁。
```

在TSL阶段，系统资源通过BS管理，BS提供的服务如下：
```
1、事件服务
2、内存管理
3、Protocol 管理
4、Protocol 使用类服务
5、驱动管理
6、Image 管理
7、ExitBootServices
```
RT提供的服务主要包括以下几个方面：

```
1、时间服务
2、读写UEFI系统变量
3、虚拟内存服务
4、其他服务
```
### 2.2 UEFI 的优点
相对BIOS，UEFI 有以下几大优势
```
1、UEFI 的开发效率
    i、绝大部分代码c编写，应用或驱动甚至可以使用c++；
    ii、通过BS 和 RT服务为OS 和OS Loader 屏蔽了底层硬件细节
2、UEFI 系统的可扩展性
    i、驱动的模块化设计
    ii、软件升级的兼容性
3、UEFI系统的性能
    i、UEFI提供了异步操作
    ii、UEFI舍弃了中断这种比较耗时的操作外部设备的方式，仅仅保留了时钟中断。
    iii、可伸缩的遍历设备的方式，启动时可以仅仅遍历启动所需的设备，从而加速系统的启动。
    iv、UEFI系统的安全性
```
### 2.3 UEFI系统的启动过程
UEFI 系统的启动遵循UEFI平台初始化（PlatformInitialization）标准。
UEFI系统从加电到关机可分为7个阶段：

```
SEC（安全验证）-> PEI（EFI前期初始化）-> DXE（驱动执行环境）-> BDS（启动设备选择）-> TSL（操作系统加载前期）-> RT（Run Time） -> AL（系统灾难恢复期）
```
![image](https://github.com/Sophie-10/Reading-Notes/blob/master/images/UEFI/1-2UEFI%E7%B3%BB%E7%BB%9F%E7%9A%847%E4%B8%AA%E9%98%B6%E6%AE%B5.png?raw=true)
图1-2
###### 1 SEC 阶段
![image](https://github.com/Sophie-10/Reading-Notes/blob/master/images/UEFI/1-3SEC%E9%98%B6%E6%AE%B5%E6%94%AF%E6%8C%81%E6%B5%81%E7%A8%8B.png?raw=true)
图1-3
```
1、SEC阶段的功能
    i、接收并处理系统启动和重启信号
    ii、初始化临时存储区域
    iii、作为可信系统的根
    iv、传递系统参数给下一阶段（即PEI）
2、SEC 阶段执行流程
    分为2个阶段：
    i、临时RAM生效之前称为Reset Vector阶段
    ii、临时RAM生效后调用SEC入口函数从而进入SEC功能区

 其中 Reset Vector 执行流程如下：
    1）、进入固件入口
    2）、从实模式转换到32位平台模式
    3）、定位固件中的BFV（Boot Firmware Volume）
    4）、定位BFV中的SEC映像
    5）、若是64位系统，从32位模式转换到64位模式
    6）、调用SEC入口函数
```
###### 2 PEI阶段
![image](https://github.com/Sophie-10/Reading-Notes/blob/master/images/UEFI/1-5%20PEI%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png?raw=true)
图1-5
```
PEI （Pre-EFI Initialization） 主要功能是为了DXE准备执行环境，将需要传递到DXE的信息组成HOB（Handoff Block）列表，最终将控制权转交到DXE手中。

    从功能上讲，氛围两部分：
    1、PEI 内核（PEI Foundation）：负责PEI基础服务和流程
    2、PEIM（PEI Module）派遣器：找出系统中所有的PEIM，并根据PEIM之间的依赖关系按吮吸执行PEIM。
        PEIM 之间的通信通过PPI（PEIM-to-PEIM Interfaces）
```

###### 3 DXE 阶段

```
DXE（Driver Execution Environment） 执行大部分系统初始化工作。
进入此阶段，内存已经可以完全被使用，因而此阶段可以进行大量的复杂工作。
![image](https://github.com/Sophie-10/Reading-Notes/blob/master/images/UEFI/1-6DXE%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png?raw=true)
图1-6
    从功能讲，分为两部分：
    1、DXE 内核：负责DXE基础服务和执行流程
    2、DXE派遣器：负责调度执行DXE驱动，初始化系统设备
    DXE提供的基础服务包括系统表、启动服务、Run Time Services。
    DXE驱动之家通过PRotocol同学。
```
当所有的Driver都执行完毕后，系统完成初始化，DXE通过EFI_BDS_ARCH_PROTOCOL找到BDS并调用BDS的入口函数，从而进入BDS阶段。
（本质讲，BDS是一种特殊的DXE阶段的应用程序）
###### 4 BDS 阶段
```
BDS（Boot Device Selection） 主要功能是执行启动策略。
    主要功能包括：
    1、初始化控制台设备
    2、加载必要的设备驱动
    3、根据系统设置加载和执行启动项
```
用户选中某个启动项（或系统进入默认的启动项）后，OS Loader启动，系统进入TSL阶段。
###### 5 TSL 阶段

```
TSL（Transient System Load）是操作系统加载器（OS Loader）执行的第一阶段，这一阶段 OS Loader 作为一个UEFI应用程序运行，系统资源仍然由UEFI内核控制。
```
当启动服务的ExitBootServices（）服务被调用后，系统进入Run Time 阶段。
###### 6 RT 阶段

```
系统进入RT（Run Time）阶段后，系统的控制权从UEFI内核转交到OS Loader手中，UEFI占用的各种资源被回收到OS Loader，仅有UEFI运行时服务保留给OS Loader 和OS使用。
```
随着OS Loader的执行，OS最终取得最多系统的控制权。
###### 7 AL 阶段

```
在RT阶段，如果系统（硬件或软件）遇到灾难性错误，系统固件需要提供错误处理和灾难恢复机制，这种机制运行在AL（After Life）阶段。UEFI 和 UEFI PI 标准都没有定义此阶段的行为和规范。
```






