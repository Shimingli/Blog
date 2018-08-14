---
title: PLC通讯的安卓App
date: 2018-02-26 16:29:26
tags: PLC
---
#### 认识中控板：华北工控 EMB-3550
基于Coretex-A17 ARM架构嵌入式All In One主板

![工控板.png](http://upload-images.jianshu.io/upload_images/5363507-919473b454fd8d08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![侧视图.png](http://upload-images.jianshu.io/upload_images/5363507-ea38ec24a0744489.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--  more  -->
![侧面图.png](http://upload-images.jianshu.io/upload_images/5363507-ded4876d0aa8ca7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![背面图](http://upload-images.jianshu.io/upload_images/5363507-1132b24da691c87c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####EMB—3550详细的参数：
◆ 采用RK3288 Coretex-A17处理器（四核）
◆ 板载2GB DDR3 高速内存，板载8GB EMMC flash
◆ 提供HDMI、VGA、LVDS多种显示端口，支持独立多显
◆ 提供5x COM，6x USB，1x USB OTG，1x LAN，1x WIFI，1x TF卡槽，Line out/Mic（板载5W双输出功放），
◆ 提供1x Mini PCIe（可支持WiFi、3G模块），1x SIM卡槽（可支持3G网络）
◆ 主板集成度高、板型紧凑、All In One 设计，尺寸仅为120mm*120mm

![详细的参数的信息.png](http://upload-images.jianshu.io/upload_images/5363507-89f4173ed2a9be2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实物图
![把板子当做一个安卓手机](http://upload-images.jianshu.io/upload_images/5363507-6288b563a0cef4af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后总结一下，我们只需要把这个设备当做手机，虽然这样，但是我在开发过程中也遇到了好多麻烦，比如说，这段的代码在手机上运行就没有什么问题，在板子上运行就直接奔溃，板子会莫名其妙的息屏，当订购板子的设备多了，会发现每一块的板子的都不一样，你说崩溃不？当然，这些都是后话了，这个板子要求两点：Root过，最好安卓的系统在5.0以下，因为那会的安卓对权限好像不是太敏感

####认识PLC：PLC控制器实质是一种专用于工业控制的计算机，其硬件结构基本上与微型计算机相同
PLC控制系统，Programmable Logic Controller，可编程逻辑控制器，专为工业生产设计的一种数字运算操作的电子装置，它采用一类可编程的存储器，用于其内部存储程序，执行逻辑运算，顺序控制，定时，计数与算术操作等面向用户的指令，并通过数字或模拟式输入/输出控制各种类型的机械或生产过程。是工业控制的核心部分。
![image.png](http://upload-images.jianshu.io/upload_images/5363507-db28b736cdd99618.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](http://upload-images.jianshu.io/upload_images/5363507-730e20aca8adaa1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 基本结构
#### 中央处理单元
中央处理单元(CPU)是PLC控制器的控制中枢。它按照PLC控制器系统程序赋予的功能接收并存储从编程器键入的用户程序和数据；检查电源、存储器、I/O以及警戒定时器的状态，并能诊断用户程序中的语法错误。当PLC控制器投入运行时，首先它以扫描的方式接收现场各输入装置的状态和数据，并分别存入I/O映象区，然后从用户程序存储器中逐条读取用户程序，经过命令解释后按指令的规定执行逻辑或算数运算的结果送入I/O映象区或数据寄存器内。等所有的用户程序执行完毕之后，最后将I/O映象区的各输出状态或输出寄存器内的数据传送到相应的输出装置，如此循环运行，直到停止运行。为了进一步提高PLC控制器的可靠性，近年来对大型PLC还采用双CPU构成冗余系统，或采用三CPU的表决式系统。这样，即使某个CPU出现故障，整个系统仍能正常运行。
#### 存储器
存放系统软件的存储器称为系统程序存储器。
存放应用软件的存储器称为用户程序存储器。
#### 电源
PLC控制器的电源在整个系统中起着十分重要得作用。如果没有一个良好的、可靠的电源系统是无法正常工作的，因此PLC的制造商对电源的设计和制造也十分重视。一般交流电压波动在+10%(+15%)范围内，可以不采取其它措施而将PLC控制器直接连接到交流电网上去。

#### 程式输入装置

负责提供操作者输入、修改、监视程式用作的功能

#### 输入输出回路

负责接收外部输入元件信号和负责接收外部输出元件信号.
###最后关于PLC，说明一下：我对这块也不熟悉，毕竟是工业上的东西，希望未来可以深入了解，上面的两张图来源于百度，可以对比发现其中的关键点是modbus通讯，我们通过modbus和PLC交互，连接的方式是LAN端口，实际上，现在在GitHub上开源的Modbus的例子特别多，所以很好的掌握上层的通讯


####认识App需要的三方依赖
```
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile 'com.android.support:appcompat-v7:23.2.1'
    compile 'com.squareup.okhttp3:okhttp-urlconnection:3.2.0'
    compile 'com.path:android-priority-jobqueue:1.1.2'
    //任务调度
    compile 'org.greenrobot:eventbus:3.0.0'
    compile 'com.kyleduo.switchbutton:library:1.4.4'
    compile 'io.reactivex:rxandroid:1.2.1'
    compile 'io.reactivex:rxjava:1.1.6'
    compile 'com.google.code.gson:gson:2.8.0'
    compile 'com.squareup.okhttp3:okhttp:3.2.0'
    compile 'com.alibaba:fastjson:1.2.21'
    //    debugCompile 'com.amitshekhar.android:debug-db:0.2.0'   //Android Debug Database 是一个用于在Android应用中调试数据库和分享参数的强大的库。
    compile 'com.amitshekhar.android:debug-db:1.0.0'//这个很nice 是一个用于在Android应用中调试数据库和分享参数的强大的库。
```
◆  okhttp:网络加载的依赖，这个不用说，搞过开发的都知道
◆（抛砖引玉）  [android-priority-jobqueue](https://link.jianshu.com/?t=https://github.com/yigit/android-priority-jobqueue)：是一个后台任务队列框架，可以对任务进行磁盘缓存，当网络恢复连接的时候继续执行任务。
######优点如下： 
便于解耦Application的业务逻辑，让你的代码更加健壮，易于重构和测试。
不处理AsyncTask的生命周期。
 Job Queue关心优先Jobs，检测网络连接，并行运行等。
可以延迟jobs。
分组jobs来确保串行执行。
默认情况下，Job Queue监控网络连接（所以你不需要担心），当设备处于离线状态，需要网络的jobs不会运行，直到网络重新连接。
◆  switchbutton：按钮
◆  Rxandroid   Rxjava：不解释
◆  debug-db：[AMIT SHEKHAR](https://github.com/amitshekhariitbhu)开源了[Android-Debug-Database](https://github.com/amitshekhariitbhu/Android-Debug-Database)，利用这个库，我们可以通过浏览器方便的查看的数据库。主要是调试用，因为有很多的数据会存到我们的数据库中
![图中的url地址可以使用浏览器打开](http://upload-images.jianshu.io/upload_images/5363507-98637c90119d4077.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![打开的效果如下](http://upload-images.jianshu.io/upload_images/5363507-0af4d354df45a40c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####串口和ModBus通信
#####串口
1、串行接口简称串口，也称串行通信接口或串行通讯接口通常指COM接口，是采用串行通信方式的扩展接口。串行接口(Serial Interface) 是指数据一位一位地顺序传送，其特点是通信线路简单，只要一对传输线就可以实现双向通信（可以直接利用电话线作为传输线），从而大大降低了成本，特别适用于远距离通信，但传送速度较慢。
2、串行接口(Serial Interface) 是指数据一位一位地顺序传送，其特点是通信线路简单，只要一对传输线就可以实现双向通信（可以直接利用电话线作为传输线），从而大大降低了成本，特别适用于远距离通信，但传送速度较慢。一条信息的各位数据被逐位按顺序传送的通讯方式称为串行通讯。串行通讯的特点是：数据位的传送，按位顺序进行，最少只需一根传输线即可完成；成本低但传送速度慢。

中控板提供了5个串口，在我的项目中使用到两个
![image.png](http://upload-images.jianshu.io/upload_images/5363507-907aec9a521b762b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####内置扫描头，记录下扫描的信息，同时保存到数据库中
#####外置扫描头，扫描提供给用户的二维码，通过扫描得到的信息，和数据库中信息相匹配，匹配到了，下发指令给PLC，PLC通知下位机做机械动作，通过联系的动作指令，把用户需要的信息反馈给用户。
![流程图01.png](http://upload-images.jianshu.io/upload_images/5363507-a32dab3d88003e0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####ModBus通信

java的modbus实现库[Git地址](https://github.com/infiniteautomation/modbus4j)

Modbus是由Modicon（现为施耐德电气公司的一个品牌）在1979年发明的，是全球第一个真正用于工业现场的总线协议。
ModBus网络是一个工业通信系统，由带智能终端的可编程序控制器和计算机通过公用线路或局部专用线路连接而成。其系统结构既包括硬件、亦包括软件。它可应用于各种数据采集和过程监控。
######Modbus具有以下几个特点：
（1）标准、开放，用户可以免费、放心地使用Modbus协议，不需要交纳许可证费，也不会侵犯知识产权。目前，支持Modbus的厂家超过400家，支持Modbus的产品超过600种。
（2）Modbus可以支持多种电气接口，如RS-232、RS-485等，还可以在各种介质上传送，如双绞线、光纤、无线等。
（3）Modbus的帧格式简单、紧凑，通俗易懂。用户使用容易，厂商开发简单。
######传输方式
在ModBus系统中有2种传输模式可选择。这2种传输模式与从机PC通信的能力是同等的。选择时应视所用ModBus主机而定，每个ModBus系统只能使用一种模式，不允许2种模式混用。一种模式是ASCII（美国信息交换码），另一种模式是RTU远程终端设备。
1、ASCII模式
当控制器设为在Modbus网络上以ASCII（美国标准信息交换代码）模式通信，一个信息中的每8个比特作为2个ASCII字符传输，如数值63H用ASCII方式时，需发送两个字节，即ASCII“6"（0110110）和ASCII”3“（0110011），ASCII字符占用的位数有7位和8位，国际通用7位为多。这种方式的主要优点是字符发送的时间间隔可达到1秒而不产生错误。
2、RTU模式
当控制器设为在Modbus网络上以RTU模式通信，在消息中的每个8Bit字节按照原值传送，不做处理，如63H，RTU将直接发送01100011。这种方式的主要优点是：数据帧传送之间没有间隔，相同波特率下传输数据的密度要比ASCII高，传输速度更快。

######代码实现
jar包，网上很多，基本上找得到，两个jar相互相成
![依赖的jar包](http://upload-images.jianshu.io/upload_images/5363507-4205e69a352897f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```    
       //"192.168.1.11"能变更的东西，尽量写活
        String plcIpAddress = AppSP.getStringValue(AppSP.PLC_IP_ADDRESS, Constant.DEFAULT_PLCIP);
        String plcPort = AppSP.getStringValue(AppSP.PLC_PORT, Constant.DEFAULT_PLC_PORT);//PLC端口号默认502
        plcNO = Integer.parseInt(AppSP.getStringValue(AppSP.PLC_NO, "1"));//默认PLC站号为1
        if (plcIpAddress.equals("") || plcPort.equals("")) {
            return;
        }

        IpParameters ipParameters = new IpParameters();
        ipParameters.setHost(plcIpAddress);
        ipParameters.setPort(Integer.parseInt(plcPort));
        if (master != null) {
            if (master.isInitialized()) {
                master.destroy();
            }
        }

        ModbusFactory modbusFactory = new ModbusFactory();
        // 参数1：IP和端口信息 参数2：保持连接激活
        master = modbusFactory.createTcpMaster(ipParameters, true);
        master.setTimeout(2000);
        //modbus重连次数
        master.setRetries(10);
        new Thread() {
            @Override
            public void run() {
                super.run();
                try {
                    master.init();
                    //连接成功，复位
                    mainHandler.post(new Runnable() {
                        @Override
                        public void run() {
                            WaittingActivity.start(getContext(), getContext().getString(R.string.msg_machine_is_all_returninng));
                        }
                    });
                    setStatus(AppStatus.NORMAL);
                } catch (ModbusInitException e) {
                    e.printStackTrace();
                    setStatus(AppStatus.PLC_CONNECT_LOST);
                } finally {
                    master.destroy();
                }
            }
        }.start();
```
写
```
  /**
     * 同步方式批量写数据到保持寄存器  功能代码：16 (0x10)
     *
     * @param master
     * @param slaveId
     * @param start
     * @param values
     */
    public static int writeHRegs(final ModbusMaster master, final int slaveId, final int start, final short[] values) {

        if (master == null) {
            return -1;
        }
        try {
            LogPrint.print_modbus(" 同步写数据到保持寄存器D", "add:" + start + "-value:" + NumberUtil.toInt(values));
            WriteRegistersRequest request = new WriteRegistersRequest(slaveId, start, values);
            WriteRegistersResponse response = (WriteRegistersResponse) master.send(request);
            if (!response.isException()) {
                return 0;
            } else {
                return -1;
            }
        } catch (ModbusTransportException e) {
            onModbusTransportException(e, start);
            e.printStackTrace();
        }
        return -1;
    }
```
读
```
  /**
     * 读取保持寄存器 功能代码 03
     *
     * @param master
     * @param slaveId
     * @param addr
     * @return
     */
    public static int readHReg(final ModbusMaster master, int slaveId, int addr) {
        try {
            LogPrint.print_modbus("同步读保持寄存器D", "add:" + addr);
            ReadHoldingRegistersRequest request2 = new ReadHoldingRegistersRequest(slaveId, addr, 1);
            ReadHoldingRegistersResponse response2 = (ReadHoldingRegistersResponse) master.send(request2);
            if (!response2.isException()) {
                return response2.getShortData()[0];
            }

        } catch (Exception e) {
            if (e instanceof ModbusTransportException) {
                onModbusTransportException((ModbusTransportException) e, addr);
            }

            e.printStackTrace();
        }
        return 0;
    }
```
####工作流程图如下，需要在新标签打开图片，放大才看的清楚些
![Modbus流程图.png](http://upload-images.jianshu.io/upload_images/5363507-cdfcc6fd6520bd0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####modbus到底是怎样的通讯的呢？
这是通讯app的控制面板，讲一个开门流程
![device-2018-02-26-175123.png](http://upload-images.jianshu.io/upload_images/5363507-fd7135a82d5b04c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击开门开关
 SwitchExecutor.openDoor();
```
         LogToFile.getInstane().write(LogToFile.SWITCH, "开门");
        Observable openDoor = Observable.create(new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                SwitchTask switchTask = new SwitchTask(DOOR, true);
                int result = switchTask.work(PLCRegAddr.OPEN_DOOR_COMP);
                switch (result) {
                    case EXEC_SUCCESS:
                        doorState = true;
                        LogToFile.getInstane().write(LogToFile.SWITCH, "开门成功");
                        break;
                    case EXEC_TIME_OUT:
                        PLCTaskBase.showMsg("开门执行超时");
                        LogToFile.getInstane().write(LogToFile.SWITCH, "开门执行超时");
                        break;
                    case WAIT_TIME_OUT:
                        PLCTaskBase.showMsg("开门等待超时");
                        LogToFile.getInstane().write(LogToFile.SWITCH, "开门等待超时");
                        break;
                }
                subscriber.onCompleted();
                SwitchMessage message = new SwitchMessage(result, DOOR);
                message.setAction(true);
                EventBus.getDefault().post(message);
            }
        });
        subscription = openDoor.observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribe(observer);

```
SwitchTask类
```
  public SwitchTask(int type, boolean value) {
        this.setaGyrate(false);
        this.type = type;
        this.value = value;
        MainApplication.setIsBusying(false);//全局变量，告知设备正在忙
    }
```
work做什么工作
public static final int OPEN_DOOR_COMP = 6034;
switchTask.work(PLCRegAddr.OPEN_DOOR_COMP);
    1、检查设备是否旋转，因为我们自己的项目中设备能够旋转，如果在旋转，就必须停止   
```
 if (aGyrate){
      //如果A轴在,停止
     GyRateTask.getInstance().stop(true);
 }

```
2、检查PLC，连接是否正常
```
        if (MainApplication.getMaster()==null){
            showMsg("主机与PLC的连接已经丢失请检查");
            return WAIT_TIME_OUT;
        }
```
3、检查设备PLC是否空闲
```
        if (MainApplication.isBusying()){
            return MACHINE_ISBUSY;
        }
```
4、设置设备正在忙，马上将要做其他的动作，要不然设备阻塞
```
        MainApplication.setIsBusying(true);
```
5、发送上机为就绪命令,参数解释，主机master ，PLC站号，指令id
```
ModbusBridge.writeCoilRegister(MainApplication.getMaster(),MainApplication.getPlcNO(), PLCRegAddr.LCS_READY,true);
```
   6、读取PLC返回消息，检查是否准备好
```
        while(!isReady){
            //超时判断
            if (waitTime>waitTimeOut){
                MainApplication.setIsBusying(false);
                return WAIT_TIME_OUT;
            }
            isReady=ModbusBridge.readCoilRegister(MainApplication.getMaster(),MainApplication.getPlcNO(), PLCRegAddr.PLC_READY);
        }

```
  7、 核心发送开门指令的代码区域
```
        ModbusBridge.writeCoilRegister(MainApplication.getMaster(),MainApplication.getPlcNO(), PLCRegAddr.CLOSE_LEFT_DOOR,false);
                    ModbusBridge.writeCoilRegister(MainApplication.getMaster(),MainApplication.getPlcNO(), PLCRegAddr.OPEN_LEFT_DOOR,true);

```
 8、读取PLC返回的消息，检查是否完成          
         
```
 boolean isComplete=false;
        while (!isComplete){
            if(execTime>execTimeOut){
                reSetRegister();
                if (aGyrate){
                    GyrateSP.putStopTime(MainApplication.getTimestamp());
                }
                MainApplication.setIsBusying(false);
                return EXEC_TIME_OUT;
            }
            isComplete=ModbusBridge.readCoilRegister(MainApplication.getMaster(),MainApplication.getPlcNO(),compAddr);
        }

```
流程图，需要在另外网页打开看的清楚一点
![App开门流程.png](http://upload-images.jianshu.io/upload_images/5363507-9d18431ccc413ad9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####以上就是一个完成的PLC开门的流程，可以看成一个水管放水的样子，我知道水管形态-PLC站号，同时知道需要往哪里放水-通讯地址，同时反馈给我们一个响应是否接收到到了，没有接收到，客服端再次的尝试，好吧这个比喻也不太明确，看做IP通讯也可以
分享点PLC的资料，如果有兴趣的话
链接: https://pan.baidu.com/s/1c3UpPL2 密码: w952

#####安卓App启动流程图
说着比较枯燥，画了一张图来说明！
![APP启动的流程图.png](http://upload-images.jianshu.io/upload_images/5363507-579ba794cbcb18f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####App和服务器的交互流程
画了一张图，只能粗略的解释下整个流程
![App和服务器的交互流程.png](http://upload-images.jianshu.io/upload_images/5363507-59debc53686f3ed0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上就是差不多整个流程，仅限分享使用








