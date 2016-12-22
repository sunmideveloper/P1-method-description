### 简介

商米P1是一个通过银联认证的POS设备，可以应用于外卖接单，打印小票，支付宝&微信支付，银行卡支付等多种场景。商米已经封装好了P1的各种底层功能，开发者可以调用这些方法对接自己的业务功能。

为了让开发者对我们封装的功能有初步的了解，我们给开发者准备了一个Demo程序，开发者可以从资源文件中下载先运行Demo，按照下面的流程操作测试该Demo中的功能，之后我们将逐一讲解Demo中的功能调用方式。

### Demo演示操作流程

1.将Demo导入AndroidStudio，其中包含两个项目两个项目，如下图所示：

![Alt SUNMI](https://github.com/sunmideveloper/P1-method-description/blob/master/imgs/1.png) 

app和sunmipaytestdemo，前者是P1金融交易相关的服务程序，这里需要开发者先在设备上安装该服务程序，之后商米会将该服务整合到ROM中，后者sunmipaytestdemo包含Demo中功能，开发者可以参照其中的代码完成自己的业务的接入调试。

2.将Demo中的app项目部署到设备中，运行该项目。

3.再将sunmipaytestdemo部署到设备中，运行该项目，界面如下所示：

![Alt SUNMI](https://github.com/sunmideveloper/P1-method-description/blob/master/imgs/2.png) 

4.点击上图的"初始化"按钮，看到上面的连接状态变成“已连接”，

5.不断点击LED灯的按钮可以不断改变设备右上角的彩色指示灯。

6.点击“蜂鸣器”按钮，可以听到滴滴的两声

7.点击“检卡”，然后将自己的银行卡按照下面的方式刷卡，注意Demo中设置的识别时间是30秒，所以请在30秒内完成刷卡的动作，完成后界面将提示读卡成功，

![Alt SUNMI](https://github.com/sunmideveloper/P1-method-description/blob/master/imgs/3.jpg) 

8.点击“信息”，界面中会显示相关的状态信息。

### 接口调用流程

下面将具体介绍接口的调用方式。

请先将com.sunmi.pay.hardware.aidl中的所有aidl文件拷贝至您项目中相同的目录包名下，记住一定是要相同的包名哦。

1.初始化
可以看到，Demo中MainActivity中的init方法中调用了ConnectPayService类中的connectPayService方法，该方法中的核心代码如下：

```
Intent intent = new Intent("sunmi.intent.action.PAY_HARDWARE");
intent.setPackage("com.sunmi.pay.hardware");
mContext.bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
```

以上就是初始化的代码，必须初始化金融支付服务才能完成P1的一系列功能操作。

2.初始化完成后，再看下设置LED灯的代码：

```
connectPayService.getDeviceProvide().getBasicProvider().ledStatusOnDevice(1, 0);
```

可以看到最终调用的是BasicProvider.aidl中的int ledStatusOnDevice(int ledIndex, int ledStatus)；该方法有两个参数 ：第一个参数：ledIndex为LED索引，设备上有4个LED灯，所以该值范围为1~4；第二个参数：ledStatus为LED状态，1表示LED灭，0表示LED亮；可以参照Demo中的顺序控制LED

3.控制蜂鸣器的代码如下：

```
connectPayService.getDeviceProvide().getBasicProvider().buzzerOnDevice(2);
```

可以看到最终调用的是BasicProvider.aidl中的int buzzerOnDevice(int times) 方法；方法中的参数为蜂鸣的次数，范围为1~10，具体请查看demo中aidl的代码注释。

4.读卡的代码如下：

```
connectPayService.getDeviceProvide().getReadCardProvider().checkCard(CARDTYPE_IC | CARDTYPE_MAG | CARDTYPE_NFC, GENERAL_READER_DEVICE, b, b.length, 30, new ReadCardCallback.Stub(){});
```

以上的checkCard中的参数说明如下：·

```
/**
   * 检卡
   * @param cardType 需要进行检卡的卡类型，CARDTYPE_MAG、CARDTYPE_IC、CARDTYPE_NFC，
   *                 支持检测一种卡或两种卡的组合，两种卡时采用与的方式传入，即CARDTYPE_MAG| CARDTYPE_IC；
   * @param deviceClass 具体的设备类别代码，如MPOS_DEVICE、READER_DEVICE；
   * @param appendData 具体的应用方数据；
   * @param appendDataLength 具体的应用方数据长度；
   * @param timeout 检卡超时时间，单位为秒；
  * @param callback 检卡回调
   */
void checkCard(int cardType, int deviceClass, in byte[] appendData, int appendDataLength, int timeout,ReadCardCallback callback);
```

最后的参数ReadCardCallback有很多的回调，具体可以参看代码中的注释，其中onCheckCardCompleted回调方法为检卡完成的回调：

通过以下简单的代码将各个参数值输出，我们可以获取到银行卡的卡号等相关信息了。

```
@Override
public void onCheckCardCompleted(int var1, int var2, String var3, String var4, Map var5) throws RemoteException {
  if (var2 == 0) {
      Log.i("sunmi", var1 + "-" +var2 + "-" + var3 + "-" + var4 );
    for(Object k : var5.keySet()){
      Log.i("sunmi", "key= "+ k + " and value= " + var5.get(k));
    }
    } else {
     }
    }
```
