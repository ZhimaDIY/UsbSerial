Based on [UsbSerial](https://github.com/felHR85/UsbSerial)
=========
Android手机的USB串口收发器. 获取更多信息请点击这里 [a more complete description](http://felhr85.net/2014/11/11/usbserial-a-serial-port-driver-library-for-android-v2-0/).
[使用UsbSerial库的APP列表](http://felhr85.net/2016/03/19/apps-and-projects-using-usbserial/)

支持的芯片
--------------------------------------
[CP210X devices](http://www.silabs.com/products/mcu/pages/usbtouartbridgevcpdrivers.aspx) Default: 9600,8,1,None,flow off

[CDC devices](https://en.wikipedia.org/wiki/USB_communications_device_class) Default 115200,8,1,None,flow off

[FTDI devices](http://www.ftdichip.com/FTProducts.htm) Default: 9600,8,1,None,flow off

[PL2303 devices](http://www.prolific.com.tw/US/ShowProduct.aspx?p_id=225&pcid=41) Default 9600,8,1,None,flow off

[CH34x devices](https://www.olimex.com/Products/Breadboarding/BB-CH340T/resources/CH340DS1.PDF) Default 9600,8,1,None,flow off

[CP2130 SPI-USB](http://www.silabs.com/products/interface/usb-bridges/classic-usb-bridges/Pages/usb-to-spi-bridge.aspx)

如何使用?
--------------------------------------
新建一个UsbSerialDevice对象
```java
UsbDevice device;
UsbDeviceConnection usbConnection;
...
UsbSerialDevice serial = UsbSerialDevice.createUsbSerialDevice(device, usbConnection); 
```

打开设备并使用它
```java
serial.open();
serial.setBaudRate(115200);
serial.setDataBits(UsbSerialInterface.DATA_BITS_8);
serial.setParity(UsbSerialInterface.PARITY_ODD);
serial.setFlowControl(UsbSerialInterface.FLOW_CONTROL_OFF); 
```

如果需要流控(目前仅支持CP201x和FTDI设备)
```java
/**
Values:
    UsbSerialInterface.FLOW_CONTROL_OFF
    UsbSerialInterface.FLOW_CONTROL_RTS_CTS 
    UsbSerialInterface.FLOW_CONTROL_DSR_DTR
**/
serial.setFlowControl(UsbSerialInterface.FLOW_CONTROL_RTS_CTS);
```

如果想处理批量的事务到 IN 端则不需要轮询. 定义一个简单的callback
```java
private UsbSerialInterface.UsbReadCallback mCallback = new UsbSerialInterface.UsbReadCallback() {

		@Override
		public void onReceivedData(byte[] arg0) 
		{
			// Code here :)
		}
		
};
```

可以参考：
```java
serial.read(mCallback);
```

CTS and DSR lines的改变 will be received in the same manner. Define a callback and pass a reference of it.
```java
private UsbSerialInterface.UsbCTSCallback ctsCallback = new UsbSerialInterface.UsbCTSCallback() {
        @Override
        public void onCTSChanged(boolean state) {
           // Code here :)
        }
    };
    
private UsbSerialInterface.UsbDSRCallback dsrCallback = new UsbSerialInterface.UsbDSRCallback() {
        @Override
        public void onDSRChanged(boolean state) {
           // Code here :)
        }
    };
    
serial.getCTS(ctsCallback);
//serial.getDSR(dsrCallback);
```



写：
```java
serial.write("DATA".getBytes()); // Async-like operation now! :)
```

Raise the state of the RTS or DTR lines
```java
serial.setRTS(true); // Raised
serial.setRTS(false); // Not Raised
serial.setDTR(true); // Raised
serial.setDTR(false); // Not Raised
```

关闭设备:
```java
serial.close();
```

推荐像如上所示的那样使用UsbSerial，但是如果你想以同步的方式执行写和读操作，可以使用这些方法:
```java
public boolean syncOpen();
public int syncWrite(byte[] buffer, int timeout)
public int syncRead(byte[] buffer, int timeout)
public void syncClose();
```


Android usb api, 当usb关闭的时候一定要重新打开
```java
UsbDevice device;
...
UsbManager manager = (UsbManager) getSystemService(Context.USB_SERVICE);
manager.openDevice(UsbDevice device)
```
如何使用SPI接口 (BETA)
--------------------------------------
最近增加了对USB到SPI设备的支持，但仍处于测试阶段。 目前仅支持CP2130芯片。

```java
UsbSpiDevice spi = UsbSpiDevice.createUsbSerialDevice(device, connection);
spi.connectSPI();
spi.selectSlave(0);
spi.setClock(CP2130SpiDevice.CLOCK_3MHz);
```
定义一个 callback

```java
private UsbSpiInterface.UsbMISOCallback misoCallback = new UsbSpiInterface.UsbMISOCallback()
    {
        @Override
        public int onReceivedData(byte[] data) {
             // Your code here :)
        }
    };
//...
spi.setMISOCallback(misoCallback);
```

```java
spi.writeMOSI("Hola!".getBytes()); // Write "Hola!" to the selected slave through MOSI (MASTER OUTPUT SLAVE INPUT)
spi.readMISO(5); // Read 5 bytes from the MISO (MASTER INPUT SLAVE OUTPUT) line. Data will be received through UsbMISOCallback
spi.writeRead("Hola!".getBytes(), 15); // Write "Hola!" and read 15 bytes synchronously
```

关闭设备
```java
spi.closeSPI();
```

Gradle
--------------------------------------
添加jitpack repo 到你的项目build.gradle文件的repositories结尾

/build.gradle
```groovy
allprojects {
	repositories {
		jcenter()
		maven { url "https://jitpack.io" }
	}
}
```

添加dependency到你的build.gradle:

/app/build.gradle
```groovy
compile 'com.github.felHR85:UsbSerial:4.5'
```

