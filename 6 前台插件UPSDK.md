# 6 前台插件UPSDK

## 6.1 接口清单

| **序号** | **类型** | **接口代码**                         | **接口名称**                                                 |
| -------- | -------- | ------------------------------------ | ------------------------------------------------------------ |
| 1.       | SDK      | pay                                  | 支付，银联在线支付                                           |
| 2.       | SDK      | addBankCard                          | 绑定银行卡                                                   |
| 3.       | SDK      | setNavigationBarTitle                | 设置标题栏标题                                               |
| 4.       | SDK      | setNavigationBarRightButton          | 设置标题栏右边按钮                                           |
| 5.       | SDK      | closeWebApp                          | 关闭当前WEB窗口                                              |
| 6.       | SDK      | showFlashInfo                        | 显示toast文言                                                |
| 7.       | SDK      | scanQRCode                           | 启动云闪付扫码                                               |
| 8.       | SDK      | chooseImage                          | 拍照或从手机相册中选图                                       |
| 9.       | SDK      | getLocationCity                      | 返回用户在首页选取的城市                                     |
| 10.      | SDK      | verifyPayPwd                         | 进入支付密码输入页面，验证支付密码输入正确与否               |
| 11.      | SDK      | getLocationGps                       | 获取用户当前GPS点，基于高德火星坐标系                        |
| 12.      | SDK      | showSharePopup                       | 显示分享弹框                                                 |
| 13.      | SDK      | chooseFileFromAlbum                  | 选择本地音视频（或图片），获取到文件基本信息                 |
| 14.      | SDK      | readAlbumData                        | 将选取的文件分块获取文件流的base64                           |
| 15.      | SDK      | startAudioRecording                  | 开始录音                                                     |
| 16.      | SDK      | stopAudioRecording                   | 停止录音                                                     |
| 17.      | SDK      | readAudioRecordingData               | 将录好的本地音频文件分块获取文件流的base64                   |
| 18.      | SDK      | startVideoRecording                  | 将录好的本地视频文件分块获取文件流的base64                   |
| 19.      | SDK      | readFaceVideoData                    | 分块获取活体检测后保存在本地的视频文件流的base64             |
| 20.      | SDK      | readFaceImageData                    | 分块获取活体检测后保存在本地的图片文件流的base64             |
| 21.      | SDK      | getScreenBrightness                  | 获取当前屏幕亮度                                             |
| 22.      | SDK      | setScreenBrightness                  | 设置屏幕亮度                                                 |
| 23.      | SDK      | monitorScreenShot                    | 监听屏幕截屏（仅限ios使用）                                  |
| 24.      | SDK      | navi                                 | 地图导航插件                                                 |
| 25.      | SDK      | openBluetoothAdapter                 | 初始化蓝牙                                                   |
| 26.      | SDK      | closeBluetoothAdapter                | 关闭蓝牙                                                     |
| 27.      | SDK      | getBluetoothAdapterState             | 获取蓝牙状态                                                 |
| 28.      | SDK      | startBluetoothDevicesDiscovery       | 开始扫描蓝牙状态                                             |
| 29.      | SDK      | stopBluetoothDevicesDiscovery        | 停止扫描蓝牙设备                                             |
| 30.      | SDK      | getBluetoothDevices                  | 获取已扫描到的设备列表                                       |
| 31.      | SDK      | connectBLEDevice                     | 连接低功耗蓝牙设备                                           |
| 32.      | SDK      | disconnectBLEDevice                  | 断开低功耗设备的蓝牙连接                                     |
| 33.      | SDK      | getConnectedBluetoothDevices         | 获取已连接的设备列表                                         |
| 34.      | SDK      | getBLEDeviceServices                 | 获取设备的服务列表                                           |
| 35.      | SDK      | getBLEDeviceCharacteristics          | 获取蓝牙设备的所有characteristic                             |
| 36.      | SDK      | writeBLECharacteristicValue          | 向蓝牙设备写数据                                             |
| 37.      | SDK      | readBLECharacteristicValue           | 向蓝牙设备读数据                                             |
| 38.      | SDK      | notifyBLECharacteristicValueChange   | 开启蓝牙设备notify提醒功能                                   |
| 39.      | SDK      | registerBluetoothDeviceFound         | 注册发现新设备监听                                           |
| 40.      | SDK      | cancelBluetoothDeviceFound           | 取消发送新设备监听                                           |
| 41.      | SDK      | registerBLEConnectionStateChange     | 注册蓝牙连接状态监听                                         |
| 42.      | SDK      | cancelBLEConnectionStateChange       | 取消蓝牙连接状态监听                                         |
| 43.      | SDK      | registerBLECharacteristicValueChange | 注册特征值变化监听                                           |
| 44.      | SDK      | cancelBLECharacteristicValueChange   | 取消特征值变化监听                                           |
| 45.      | SDK      | registerBluetoothAdapterStateChange  | 注册蓝牙状态变化监听                                         |
| 46.      | SDK      | cancelBluetoothAdapterStateChange    | 取消蓝牙状态变化监听                                         |
| 47.      | SDK      | openBluetoothSetting                 | 跳转到手机系统的蓝牙设置                                     |
| 48.      | SDK      | getHCEState                          | 判断当前设备是否支持HCE能力                                  |
| 49.      | SDK      | startHCE                             | 调用startHCE初始化本机NFC模块                                |
| 50.      | SDK      | stopHCE                              | 调用stopHCE停止手机的NFC模块                                 |
| 51.      | SDK      | onHCEMessage                         | 调用onHCEMessage监听芯片响应的消息事件中返回。               |
| 52.      | SDK      | sendHCEMessage                       | 调用 sendHCEMessage发送写卡数据并在成功回调中接收芯片响应的消息 |
| 53.      | SDK      | openNFCSetting                       | 打开NFC设置页面                                              |

 

## 6.2 UPSDK使用步骤

**步骤一：引入****JS****文件**

在需要调用JS接口的页面引入如下JS文件（仅支持https）： 

https://open.95516.com/s/open/js/upsdk.js

 

注意：

Ø upsdk.js文件依赖Zepto或Jquery。

Ø Zepto的版本要求1.0及以上，Jquery的版本要求1.4及以上（建议用最新的Zepto及Jquery）。

 

**步骤二：通过****config****接口注入权限验证配置**

所有需要使用UPSDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次，如果跳转的页面无需使用插件，则无需config，否则需要重新执行config）。



建议接入方在开发联调时，打开debug: true 开关，UPSDK只有在开关打开时才会输出状态信息，帮助开发者定位错误。 请务必在最终生产版本关闭此开关。

 

**步骤三：配置信息验证**

通过ready接口处理成功验证



通过error接口处理失败验证



 

## 6.3 支付接口<pay>

upsdk.pay({
   tn: '支付流水号Ticket Number.',
   success: function(){
     // 支付成功，开发者执行后续操作
   },
   fail: function(err){
     // 支付失败，err.msg 是失败原因描述，比如TN号不合法，或者用户取消了交易等等
   }
 });

前台支付对接，交易TN号获取方式参见如下URL：

https://open.unionpay.com/tjweb/acproduct/APIList?acpAPIId=393&&apiservId=450

整个支付流程详解请参见附录五-支付流程详解。

## 6.4 拍照或从手机相册中选图接口<chooseImage>

upsdk.chooseImage({
   maxWidth: '目标图片宽度，默认500',   //可选。
   maxHeight: '目标图片高度，默认1000',  //可选。
   sourceType: '1|2|3，仅允许拍照|仅允许从手机相册中选图|拍照或从手机相册中选图都支持， 默认为3',  //可选
   success: function (data) {
     if (data.base64) {
       // 目标图片采用base64编码
     }
   }
 });

此接口还会返回选择的图片类型，如jpg|png|gif

## 6.5 设置页面标题接口<setNavigationBarTitle>

upsdk.setNavigationBarTitle({
   title: '设置云闪付标题'
 });

## 6.6 设置标题栏右边按钮接口<setNavigationBarRightButton>

upsdk.setNavigationBarRightButton({
   // title和image可任选其一，同时有的话，title优先
   title: '标题栏文字',
   image: ' 按钮图片绝对路径',­­­­­­
   handler: function(){
     // 用户点击标题按钮以后回调函数
   }
 });

支持文字、图片

## 6.7 调用云闪付的扫码功能接口<scanQRCode>

upsdk.scanQRCode({
   scanType: ["qrCode","barCode"],
   success: function(result){
     alert('Scan result = ' + result);
   }
 });

## 6.8 绑定银行卡接口<addBankCard>

upsdk.addBankCard({
   success: function(){
     // 绑卡成功
   },
   fail: function(){
     // 绑卡失败或者用户取消
   }
   sence: '场景号' //可选参数，用来控制添加特定类型的银行卡，例如只添加信用卡
 });

## 6.9 Toast文言显示接口<showFlashInfo>

upsdk.showFlashInfo({
   msg: '绑定银行卡成功'
 });

## 6.10 关闭当前WEB窗口接口<closeWebApp>

upsdk.closeWebApp();

## 6.11 返回用户在首页选取的城市<getLocationCity>

upsdk.getLocationCity({
   success: function(cityCd){

​     //插件返回的城市代码cityCd 符合全国行政区划代码表标准，比如广州市为440100  

});

## 6.12 用户通过输入支付密码授权<verifyPayPwd>

upsdk.verifyPayPwd({
   transNo: '后台获取的支付密码验证流水号', 

bizType: '业务类型编号',
   success: function() {
     // 支付密码验证通过
   }
 });

transNo：通过后台接口“获取支付密码验证流水号<verifyPwdSeqId>”获取

bizType：业务类型，请与云闪付业务方确定

## 6.13 获取用户当前位置经纬度<getLocationGps>

upsdk.getLocationGps({
   success: success,
   fail: fail
 });

成功返回对象，里面含有属性latitude 和longitude，基于高德地图坐标系

## 6.14 云闪付分享插件<showSharePopup>

upsdk.showSharePopup({
   title: '银联云闪付随机立减大优惠～！',
   desc: '我刚刚使用银联云闪付，省了30元，大家快来使用吧。 ',
   shareUrl: 'https://youhui.95516.com',
   picUrl: 'https://youhui.95516.com/web/image/mchnt/coupon/Z00000000138804_logo_list_web.jpg'
 });

picUrl 选填，默认显示银联云闪付图标

## 6.15 获取本地音视频或图片<chooseFileFromAlbum & readAlbumData>

**步骤一：选择本地音视频或图片，获取文件基本信息**

upsdk.chooseFileFromAlbum({
   maxSize:'1024',  //最大上传的文件大小（单位byte），若选择的文件大于该值进入失败回调。建议值不要超过100m（即100*1024*1024）
   sourceType:'00',  //00:仅支持视频文件，例如mp4、mov等。01：仅支持图片文件，例如png、jpg等。02：支持视频+图片文件。
   success:function(data){  

​       // 成功返回{url:'视频地址',size:'视频大小',filename:'视频文件名，例如123.mp4'}

}, 
   fail: function(err){

​         // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​      // {code:'01',msg:'内部错误'}

​      // {code:'02',msg:'权限失败'}

​      // {code:'03',msg:'用户选择的文件超过最大值'}

​      // {code:'04',msg:'其他错误'}

}**
** });

 

**步骤二：将选取的文件分块，通过以下插件分块获取文件流的****base64**

upsdk.readAlbumData({
   url: 'url',  //即步骤一成功回调获取视频地址url
   bufferSize: '1024',  //分块大小，单位byte，最大不可以超过512kb（即512*1024）
   fromOffset: '0',   //每块的开始位置，第一块为0，第二块为1*bufferSize，依次类推

success:function(data){  

​       // 成功返回{data:'文件块的base64',isFinished:'1'}

​       // isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕

}, 
 fail: function(err){ 

​         // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​       // {code:'01',msg:'内部错误'}

​      // {code:'02',msg:'权限失败'}

​      // {code:'03',msg:'用户选择的文件超过最大值'}

​      // {code:'04',msg:'其他错误'}

  }**
** });

## 6.16 录音并获取录音文件< startAudioRecording & stopAudioRecording & readAudioRecordingData>

**步骤一：开始录音，获得成功回调后，可以选择主动停止录音，或者等待最大录音时间后自动停止。**

upsdk.startAudioRecording({
   maxTime:'10',  //单位为秒，必选参数，允许用户录制的最大时间
   success:function(){ 

​       //成功回调，此处可选择主动停止录音，或者等待maxTime

}, 
   fail: function(err){

​         // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​      // {code:'01',msg:'内部错误'}

​      // {code:'02',msg:'权限失败'}

​      // {code:'03',msg:'用户选择的文件超过最大值'}

​      // {code:'04',msg:'其他错误'}

​    }**
** });

upsdk.stopAudioRecording({
   success:function(){  

​       // 成功回调，此处可获取已录好的本地音频文件

}, 
   fail: function(err){

​         // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​      // {code:'01',msg:'内部错误'}

​      // {code:'02',msg:'权限失败'}

​      // {code:'03',msg:'用户选择的文件超过最大值'}

​      // {code:'04',msg:'其他错误'}

​    }**
** });

 

**步骤二：将录好的本地音频文件分块，通过以下插件分块获取文件流的****base64****。【注意，该插件应该在停止录音插件成功回调中使用，或者等待最大录音时间****maxTime****后才能使用】**

upsdk.readAudioRecordingData({
   bufferSize: '1024', //分块大小，单位byte，最大不可以超过512kb（即512*1024）
   fromOffset: '0',  //每块的开始位置，第一块为0，第二块为1*bufferSize，依次类推

success:function(data){ 

​        // 成功返回{data:'文件块的base64',isFinished:'1'}

​       // isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕

}, 
   fail: function(err){  

​       // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​      // {code:'01',msg:'内部错误'}

​      // {code:'02',msg:'权限失败'}

​      // {code:'03',msg:'用户选择的文件超过最大值'}

​      // {code:'04',msg:'其他错误'}

  }**
** });

备注：始终保存最后一次录音的文件，仅在卸载APP后才会清除。

## 6.17 录视频并获取视频文件<startVideoRecording & readVideoRecordingData>

**步骤一：开始录视频，到达最大录制时间，视频自动录制结束。用户录制完视频后调用成功回调。**

upsdk.startVideoRecording({
   maxTime: '10',  //单位为秒，必选参数，允许用户录制的最大时间
   success:function(){ 

​       // 成功回调，此时用户已完成录制视频，可获取录制的视频文件

}, 
   fail: function(err){

​         // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​      // {code:'01',msg:'内部错误'}

​      // {code:'02',msg:'权限失败'}

​      // {code:'03',msg:'用户选择的文件超过最大值'}

​      // {code:'04',msg:'其他错误'}

}**
** });

 

​    **步骤二：将录好的本地视频文件分块，通过以下插件分块获取文件流的****base64****。【注意，该插件应该在开始录视频插件成功回调中使用。】**

upsdk.readVideoRecordingData({
   bufferSize: '1024',  //分块大小，单位byte，最大不可以超过512kb（即512*1024）
   fromOffset: '0',   //每块的开始位置，第一块为0，第二块为1*bufferSize，依次类推

success:function(data){ 

​       // 成功返回{data:'文件块的base64',isFinished:'1'}

​       // isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕

}, 
   fail: function(err){ 

​         // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​      // {code:'01',msg:'内部错误'}

​      // {code:'02',msg:'权限失败'}

​      // {code:'03',msg:'用户选择的文件超过最大值'}

​      // {code:'04',msg:'其他错误'}

  }**
** });

备注：始终保存最后一次录音的文件，仅在卸载APP后才会清除。

## 6.18 获取活体检测后保存在本地的视频和图片<readFaceVideoData& readFaceImageData>

获取活体检测的视频（分块获取，视频格式为mp4）

upsdk.readFaceVideoData({

success:function(data){ 

​       // 成功返回{data:'文件块的base64',isFinished:'1'}

​       // isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕

}, 
   fail: function(err){ 

​         // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​      // {code:'01',msg:'内部错误'}

​      // {code:'07', msg:'文件不存在'}

  }**
** });

 

获取活体检测的图片（分块获取，图片格式为jpg）

upsdk.readFaceImageData({

success:function(data){ 

​       // 成功返回{data:'文件块的base64',isFinished:'1'}

​       // isFinished：0代表文件还有块未获取，1代表文件所有块已获取完毕

}, 
   fail: function(err){  

​         // 失败回调{code:'失败码',msg:'失败原因描述'}

​       // {code:'00',msg:'参数错误'}

​      // {code:'01',msg:'内部错误'}

​      // {code:'07', msg:'文件不存在'}

  }**
** });

## 6.19 获取当前屏幕亮度<getScreenBrightness>

upsdk.getScreenBrightness({

success:function(data){ 

// 成功回调 {data.brightness} 

}, 
 fail: function(msg){ 

// 失败回调

}

});

## 6.20 设置屏幕亮度<setScreenBrightness>

upsdk.setScreenBrightness({
   brightness**:** '0.2'**,**   // 屏幕亮度值，范围取值0-1。精确到小数点后一位

success:function(data){ 

// 成功回调 {data.brightness} 

}, 
   fail: function(msg){  

// 失败回调

}**
** });

## 6.21 监听屏幕截屏（仅限ios使用）<monitorScreenShot>

upsdk.monitorScreenShot({

success:function(){ 

// 成功回调，表明用户已经进行了截屏

}

});

备注：通常只在需要监听的页面调用该插件打开监听。

## 6.22 地图导航插件<navi>

upsdk.navi({
   sLat**:**'31.23958'**,**    // 起点纬度

sLon**:**'121**.**499763'**,**   // 起点经度

  sName**:**'上海东方明珠', // 起点名称

  dLat**:**'39.917854',    // 终点维度

  dLon**:**'116.397006',   // 终点经度

  dName**:**'北京故宫',   // 终点名称

success:function(){ 

// 成功回调 

}, 
   fail: function(msg){  

// 失败回调

}
 });

 

## 6.23  蓝牙插件

### 6.23.1初始化蓝牙<openBluetoothAdapter>

**­­­­ upsdk**.openBluetoothAdapter({

**success**:**function(data){** 

// 成功回调 {"isSupportBLE": "yes"} 支持BLE，不区分大小写

// 成功回调 {"isSupportBLE": "no"} 不支持BLE，不区分大小写

**}**, 
   fail:**function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

注意：只有初始蓝牙成功后才能正常执行后续操作蓝牙功能

### 6.23.2关闭蓝牙<closeBluetoothAdapter>

**­­­­ upsdk**.closeBluetoothAdapter({

**success**:**function(){** 

// 成功回调 

**}**, 
   fail: **function(){** 

  // 失败回调

**}
** });

 

### 6.23.3获取蓝牙状态<getBluetoothAdapterState>

**­­­­ upsdk**.getBluetoothAdapterState({

**success**:**function(data){** 

// 成功回调 {"available":"yes","discovering":"no"} 

// available的值为yes 或者 no，不区分大小，表明当前蓝牙是否可用

// discovering 的值为 yes 或者 no，不区分大小，表明是否在扫描蓝牙中

**}**, 
   fail: **function(){** 

  // 失败回调

**}
** });

### 6.23.4开始扫描蓝牙设备<startBluetoothDevicesDiscovery>

**­­­­ ­­­­ upsdk**.startBluetoothDevicesDiscovery({

**services: [****‘具体的****uuid****’****]****，**  **//** **可选，蓝牙设备主** **service** **的** **uuid** **列表**

**allowDuplicatesKey: ‘yes’,  //** **可选，值为****yes****或****no****，是否允许重复上报同一设备**

**success**:**function(){** 

// 成功回调 

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

注意：可选字段不赋予值的时候不要上送

 

### 6.23.5停止扫描蓝牙设备<stopBluetoothDevicesDiscovery>

```
­­­­ ­­­­ upsdk.stopBluetoothDevicesDiscovery({
```

**services: [****‘具体的****uuid****’****]****，**  **//** **可选，蓝牙设备主** **service** **的** **uuid** **列表**

**allowDuplicatesKey: ‘yes’,  //** **可选，值为****yes****或****no****，是否允许重复上报同一设备**

**success**:**function(){** 

// 成功回调 

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

注意：可选字段不赋予值的时候不要上送

 

### 6.23.6获取已扫描到的设备列表<getBluetoothDevices>

**­­­­ ­­­­ upsdk**.getBluetoothDevices({

**success**:**function(data){** 

// 成功回调 {devices: [{device}]}，device参数如下表所示

**}**, 
   fail: **function(){** 

  // 失败回调 

**}
** });

![img](file:///C:\Users\ZHIPEN~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.gif)

 

### 6.23.7连接低功耗蓝牙设备<connectBLEDevice>

**­­­­ upsdk**.connectBLEDevice({

**deviceId:’’, //** **必填，设备****id****，可从获取已扫描到的设备列表****6****）中获取该值，并进行连接**

**success**:**function(){** 

// 成功回调 

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.8断开低功耗蓝牙设备<disconnectBLEDevice>

**­­­­ upsdk**.disconnectBLEDevice({

**deviceId:’’, //** **必填，设备****id****，可从获取已扫描到的设备列表中获取该值，并进行连接**

**success**:**function(){** 

// 成功回调 

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.9获取已连接的设备列表<getConnectedBluetoothDevices>

注意:­­­­只有当监听蓝牙连接状态<registerBLEConnectionStateChange>执行了回调，且连接状态connected为yes时，才可获取到设备列表

可选字段不赋予值的时候不要上送

**­­­­ upsdk**.getConnectedBluetoothDevices({

**services: [****‘具体的****uuid****’****]****，**  **//** **可选，蓝牙设备主** **service** **的** **uuid** **列表**

**success**:**function(data){** 

// 成功回调 {devices: [{device}]}，device参数同 6) 

**}**, 
   fail: **function(){** 

  // 失败回调 

**}
** });

### 6.23.10获取设备的服务列表<getBLEDeviceServices>

**­­­­ upsdk**.getBLEDeviceServices ({

**deviceId:’’, //** **必填，设备****id****，需从获取已连接的设备列表** **9****）中获取该值**

**success**:**function(data){** 

// 成功回调 {"services": [{service}]}，service参数如下表所示

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

![img](file:///C:\Users\ZHIPEN~1\AppData\Local\Temp\msohtmlclip1\01\clip_image004.gif)

### 6.23.11获取蓝牙设备的所有Cahracteristic<getBLEDeviceCharacteristics>

```
­­­­ upsdk.getBLEDeviceCharacteristics({
```

**deviceId:’’, //** **必填，设备****id****，需从获取已连接的设备列表** **9****）中获取该值**

**serviceId:’’, //** **必填，服务****id****，需从获取设备的服务列表** **10****）中获取该值**

**success**:**function(data){** 

// 成功回调 {"characteristics": [{characteristic}]}，characteristic参数如下表所示

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

![img](file:///C:\Users\ZHIPEN~1\AppData\Local\Temp\msohtmlclip1\01\clip_image006.gif)

注意：确保该接口能正常获取返回的码表信息，如不能将影响后续系列操作；

 

### 6.23.12向蓝牙设备写数据<writeBLECharcateristicValue>

注意：­­­­调用该插件时，需characteristicId对应的属性有Write权限 

**upsdk**.writeBLECharacteristicValue ({

**deviceId:’’, //** **必填，设备****id****，需从获取已连接的设备列表** **9****）中获取该值**

**serviceId:’’, //** **必填，服务****id****，需从获取设备的服务列表** **10****）中获取该值**

**characteristicId:’’, //** **必填，特征****id****，需从获取设备的特征列表** **11****）中获取该值**

**value:’’, //** **必填，写的数据** 

**success**:**function(){** 

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

### 6.23.13向蓝牙设备读数据<readBLECharacteristicValue>

注意：­­­­调用该插件时，需characteristicId对应的属性有Read权限 

**upsdk**.readBLECharacteristicValue({

**deviceId:’’, //** **必填，设备****id****，需从获取已连接的设备列表** **9****）中获取该值**

**serviceId:’’, //** **必填，服务****id****，需从获取设备的服务列表** **10****）中获取该值**

**characteristicId:’’, //** **必填，特征****id****，需从获取设备的特征列表** **11****）中获取该值**

**success**:**function(data){** 

  // 成功回调{characteristic: {characteristicId:’’, serviceId:’’, value:’’}}

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

### 6.23.14开启蓝牙设备notify提醒功能<notifyBLECharacteristicValueChange>

注意：­­­­调用该插件时，需characteristicId对应的属性有Notify或Indicate权限 

**upsdk**.notifyBLECharacteristicValueChange({

**deviceId:’’, //** **必填，设备****id****，需从获取已连接的设备列表** **9****）中获取该值**

**serviceId:’’, //** **必填，服务****id****，需从获取设备的服务列表** **10****）中获取该值**

**characteristicId:’’, //** **必填，特征****id****，需从获取设备的特征列表** **11****）中获取该值**

**state:’’, //****选填，预留参数，暂无作用；只能开启特征值支持的模式；**

​       **//****如果特征值支持****notify****和****indicate****两种模式，则使用****notify****模式**

**success**:**function(){** 

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.15注册发现新设备监听<registerBluetoothDeviceFound>

**upsdk**.registerBluetoothDeviceFound ({

**callback**:**function(device){** 

  // 异步，扫描到设备时会执行该回调,

  // 并返回参数device = {

  //              name:’’,

  //             localName:’’,

  //             deviceId:’’,

  //             RSSI:’’,

  //             manufacturerData:’’

 //             }

**}**, 

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.16取消发现新设备监听<cancelBluetoothDeviceFound>

**upsdk**.cancelBluetoothDeviceFound({

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.17注册蓝牙连接状态监听<registerBLEConnectionStateChange>

**upsdk**.registerBLEConnectionStateChange ({

**callback**:**function(state){** 

  // 异步，设备连接状态发生改变时会执行该回调,

  // 并返回参数state = {

  //             deviceId:’’, // 设备id

  //            connected:’’ //值为yes表示已连接，值为no表示未连接

  //           }

**}**, 

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.18取消蓝牙连接状态监听<cancelBLEConnectionStateChange>

**upsdk**.cancelBLEConnectionStateChange({

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.19注册特征值变化监听<registerBLECharacteristicValueChange>

**upsdk**.registerBLECharacteristicValueChange({

**callback**:**function(characteristic){** 

  // 异步，特征值发生变化时会执行该回调,

  // 并返回参数characteristic = {

  //     deviceId:’’,

  //     serviceId:’’,

  //     characteristicId:’’,

  //     value:’’

  //  }

**}**, 

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.20取消特征值变化监听<cancelBLECharacteristicValueChange>

**upsdk**.cancelBLECharacteristicValueChange({

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.21注册蓝牙状态变化监听<registerBluetoothAdapterStateChange>

**upsdk**.registerBluetoothAdapterStateChange({

**callback**:**function(data){** 

  // 异步，设备从未初始化->初始化 或 从初始化->未初始化 等状态发生变化时，执行回调

  // data = {state:’’}  state取值说明见下表

**}**, 

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

![img](file:///C:\Users\ZHIPEN~1\AppData\Local\Temp\msohtmlclip1\01\clip_image008.gif)

 

### 6.23.22取消取消蓝牙变化监听<cancelBluetoothAdapterStateChange>

**upsdk**.cancelBluetoothAdapterStateChange({

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(data){** 

  // 失败回调 {code:’’,msg:’’}

**}
** });

 

### 6.23.23跳转到手机系统的蓝牙设置<openBluetoothSetting>

**upsdk**.openBluetoothSetting({

**success**:**function(){ //** **插件执行成功**

**}**, 
   fail: **function(){** 

**}
** });

 

 

 

 

 

 

 

 

**备注：错误码说明**

![img](file:///C:\Users\ZHIPEN~1\AppData\Local\Temp\msohtmlclip1\01\clip_image010.gif)

 

## 6.24  NFC插件（HCE模式）

### 6.24.1判断当前设备是否支持HCE能力<getHCEState>

**upsdk**.getHCEState ({

**success**:**function(data){ //** **插件执行成功**

  **//**

**}**, 
   fail: **function(){** 

**}
** });

成功回调的响应：

| code | msg                                  |
| ---- | ------------------------------------ |
| 0000 | ok                                   |
| 1001 | 当前设备不支持NFC                    |
| 1002 | 当前设备支持NFC，但系统NFC开关未开启 |
| 1003 | 当前设备支持NFC，但不支持HCE         |
| 1004 | 未设置云闪付为默认NFC应用            |

 

 

### 6.24.2调用startHCE初始化本机NFC模块<startHCE>

**upsdk**.**startHCE** ({

 

**aidList**:**""**,**//** **必填，需要注册到系统的** **AID** **列表**

**success**:**function(data){ //** **插件执行成功**

  **//**

**}**, 
   fail: **function(){** 

**}
** });

```
注释：aidList 参数类型为string 多个AID 请用“,”隔开,AID长度限制是10-32位由0-9和A-F的字符组成实例 "A000000010,A123456789"
```

 

| code | msg                 |
| ---- | ------------------- |
| 0000 | ok                  |
| 1101 | 注册AID失败         |
| 1102 | AID列表参数格式错误 |
| 1103 | 输入参数错误        |

 

### 6.24.3调用stoptHCE停止手机NFC模块<stoptHCE>

**upsdk.stopHCE(**{

**success**:**function(data){ //** **插件执行成功**

  **//**

**}**, 
   fail: **function(){** 

**}
** });

| code | msg          |
| ---- | ------------ |
| 0000 | ok           |
| 1401 | 输入参数错误 |
| 1402 | 注销AID失败  |

 

 

### 6.24.4调用onHCEMessage监听芯片响应的消息事件中返回<onHCEMessage>

**upsdk.**onHCEMessage**(**{

 

 

**success**:**function(data){ //** **插件执行成功**

  **//**

**}**, 
   fail: **function(){** 

**}
** });

注释：该方法用于监听NFC设备发来的指令，调用该方法需要调用startHCE成功后才可以调用

| code | msg          |
| ---- | ------------ |
| 0000 | ok           |
| 1301 | 发送指令失败 |

 

### 6.24.5调用sendHCEMessage发送写卡数据并在成功回调中接口芯片响应的消息<sendHCEMessage>

```
upsdk.sendHCEMessage({
    data:"",// 必填，命令数据，类型是string，内容是01组成的2进制数据
```

**success**:**function(data){ //** **插件执行成功**

  **//**

**}**, 
   fail: **function(){** 

**}
** });

### 6.24.6打开NFC设置页面<openNFCSetting>

```
upsdk.openNFCSetting({
 
```

**success**:**function(data){ //** **插件执行成功**

  **//**

**}**, 
   fail: **function(){** 

**}
** });