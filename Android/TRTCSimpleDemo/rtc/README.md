## 适用场景
TRTC 支持四种不同的进房模式，其中视频通话（VideoCall）和语音通话（VoiceCall）统称为[通话模式](https://cloud.tencent.com/document/product/647/32169)，视频互动直播（Live）和语音互动直播（VoiceChatRoom）统称为[直播模式](https://cloud.tencent.com/document/product/647/35428)。

通话模式下的 TRTC，支持单个房间最多300人同时在线，支持最多50人同时发言。适合1对1视频通话、300人视频会议、在线问诊、远程面试、视频客服、在线狼人杀等应用场景。

## 原理解析
TRTC 云服务由两种不同类型的服务器节点组成，分别是“接口机”和“代理机”：

- **接口机**
这些节点都采用最优质的线路和高性能的机器，善于处理端到端的低延时连麦通话，单位时长计费较高。

- **代理机**
这些节点都采用普通的线路和性能一般的机器，善于处理高并发的拉流观看需求，单位时长计费较低。

在通话模式下，TRTC 房间中的所有用户都会被分配到接口机上，相当于每个用户都是“主播”，每个用户随时都可以发言（最高的上行并发限制为50路），因此适合在线会议等场景，不过单个房间的人数限制为300人。

![](https://main.qcloudimg.com/raw/b88a624c0bd67d5d58db331b3d64c51c.gif)

## 示例代码
访问 [Github](https://github.com/tencentyun/TRTCSDK/tree/master/Android/TRTCSimpleDemo) 即可获取本文档相关的实例代码。
![](https://main.qcloudimg.com/raw/6b22e023d733cd16976b00c22dd53b61.png)

## 使用步骤
<span id="step1"> </span>
### 步骤1：集成 SDK 到项目中
可以选择如下任意一种方式将 **TRTC SDK** 集成到项目中。
#### 自动加载（aar）
TRTC SDK 已经发布到 jcenter 库，您可以通过配置 gradle 自动下载更新。
只需要用 Android Studio 打开 TRTCSimpleDemo 工程，然后通过简单的三个步骤修改 app/build.gradle 文件，就可以完成 SDK 集成

1. 添加 SDK 依赖
在 dependencies 中添加 TRTCSDK 的依赖。
```
dependencies {
  compile 'com.tencent.liteav:LiteAVSDK_TRTC:latest.release'
}
```
2. 指定 App 使用架构
在 defaultConfig 中，指定 App 使用的 CPU 架构(目前 TRTC SDK 支持 armeabi ， armeabi-v7a 和 arm64-v8a) 。
```
 defaultConfig {
      ndk {
          abiFilters "armeabi", "armeabi-v7a", "arm64-v8a"
      }
  }
```
3. 同步 SDK
单击 Sync Now 按钮，如果您的网络连接 jcenter 没有问题，很快 SDK 就会自动下载集成到工程里。

#### 下载 ZIP 包手动集成
如果您的网络连接 jcenter 有问题，可以考虑在下载页面直接下载 [ZIP 压缩包](https://cloud.tencent.com/document/product/647/32689)，并按照集成文档 [手动集成](https://cloud.tencent.com/document/product/647/32175#.E6.96.B9.E6.B3.95.E4.BA.8C.EF.BC.9A.E6.89.8B.E5.8A.A8.E4.B8.8B.E8.BD.BD.EF.BC.88aar.EF.BC.89) 到您的工程中。


<span id="step2"> </span>
### 步骤2：在工程文件中配置 App 权限
在 **AndroidManifest.xml** 文件中添加摄像头和麦克风，以及网络的申请权限

```
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.BLUETOOTH" />

<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
```


<span id="step3"> </span>
### 步骤3：初始化 SDK 实例并监听事件回调

1. 使用 [sharedInstance()](https://cloud.tencent.com/document/product/647/32267) 接口创建 `TRTCCloud` 实例。
2. 设置 setListener 属性注册事件回调，并监听相关事件和错误通知。
```
// 创建 trtcCloud 实例
mTRTCCloud = TRTCCloud.sharedInstance(getApplicationContext());
mTRTCCloud.setListener(new TRTCCloudListener());
```
监听
```
// 错误通知监听，错误通知意味着 SDK 不能继续运行
@Override
public void onError(int errCode, String errMsg, Bundle extraInfo) {
    Log.d(TAG, "sdk callback onError");
    if (activity != null) {
        Toast.makeText(activity, "onError: " + errMsg + "[" + errCode+ "]" , Toast.LENGTH_SHORT).show();
        if (errCode == TXLiteAVCode.ERR_ROOM_ENTER_FAIL) {
            activity.exitRoom();
        }
    }
}
```

<span id="step4"> </span>
### 步骤4：组装进房参数 TRTCParams
在调用 [enterRoom()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#abfc1841af52e8f6a5f239a846a1e5d5c) 接口时需要填写一个关键参数 [TRTCParams](http://doc.qcloudtrtc.com/group__TRTCCloudDef__android.html#a674b3c744a0522802d68dfd208763b59)，它包含如下四个必填的字段：sdkAppId、userId、userSig 和 roomId。

| 参数名称 | 填写示例 | 字段类型 | 补充说明 |
|---------|---------|---------|---------|
| sdkAppId | 1400000123 | 数字 | 应用ID，您可以在 [控制台](https://console.cloud.tencent.com/trtc/app) >【应用管理】>【应用信息】中查找到。 |
| userId | test_user_001 | 字符串 | 只允许包含大小写英文字母（a-zA-Z）、数字（0-9）及下划线和连词符。 |
| userSig | eJyrVareCeYrSy1SslI... | 字符串 | 基于 userId 可以计算出 userSig，计算方法请参见 [UserSig 计算](https://cloud.tencent.com/document/product/647/17275) 。|
| roomId | 29834 | 数字 | 默认不支持字符串类型的房间号，字符串类型的房间号会拖慢进房速度，如果您确实需要支持字符串类型的房间号，请通过工单联系我们。 |

>! TRTC 同一时间不支持两个相同的 userId 进入房间，否则会相互干扰。

<span id="step5"> </span>
### 步骤5：创建并进入房间
1. 调用 [enterRoom()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#abfc1841af52e8f6a5f239a846a1e5d5c) 即可加入 TRTCParams 参数中 `roomId` 所指定的音视频房间。如果该房间不存在，SDK 会自动创建一个以 `roomId` 为房间号的新房间。
2. 指定 **appScene** 参数，如果是视频通话，请指定为 `TRTC_APP_SCENE_VIDEOCALL`，如果是语音通话，则指定为 `TRTC_APP_SCENE_AUDIOCALL`。
3. 如果进房成功，SDK 会回调 `onEnterRoom（result）` 事件，参数 `result` 大于0时代表进房成功，数值表示加入房间所消耗的时间，单位为毫秒（ms）；当 `result` 小于0时代表进房失败，数值表示进房失败的错误码。

```
public void enterRoom() {
    TRTCCloudDef.TRTCParams trtcParams = new TRTCCloudDef.TRTCParams();
    trtcParams.sdkAppId = sdkappid;
    trtcParams.userId = userid;
    trtcParams.roomId = usersig;
    trtcParams.userSig = 908;
    mTRTCCloud.enterRoom(trtcParams, TRTC_APP_SCENE_VIDEOCALL);
}

@Override
public void onEnterRoom(long result) {
    if (result > 0) {
        toastTip("进房成功，总计耗时[\(result)]ms")
    } else {
        toastTip("进房失败，错误码[\(result)]")
    }
}
```

>! 
>1. 请根据应用场景选择合适的 appScene 参数，使用错误可能会导致卡顿率或画面清晰度不达预期。
>2. 如进房失败，SDK 同时还会回调 `onError` 事件，参数：`errCode`（[错误码](https://cloud.tencent.com/document/product/647/38307)）、`errMsg`（错误原因）、`extraInfo`（保留参数）。
>3. 如果已在某一个房间中，则必须先调用 `exitRoom()` 退出当前房间之后，才能进入下一个房间。

<span id="step6"> </span>
### 步骤6：订阅远端的音视频流
SDK 支持自动订阅和手动订阅两种模式，接下来我们分别对其进行介绍：

#### 自动订阅模式（默认）
这是 SDK 默认的订阅模式。在自动订阅模式下，当 SDK 进入某个房间之后，会自动接收房间中其他用户的音频流，从而达到最佳的秒开效果：
1. 当房间中有其他用户在上行音频数据时，您会收到 [onUserAudioAvailable()](http://doc.qcloudtrtc.com/group__TRTCCloudListener__android.html#ac474bbf919f96c0cfda87c93890d871f) 事件通知，SDK 会自动播放这些远端用户的声音。
2. 您可以通过 [muteRemoteAudio(userId, true)](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a8d8b8edf120036d4049cc3639a1ce81f) 屏蔽某一个 userId 的音频数据，也可以通过 [muteAllRemoteAudio(true)](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a5b63c0796404b80323ae67aafe0384ba) 屏蔽所有远端用户的音频数据，之后 SDK 不再继续拉取这些远端用户的音频数据。
3. 当房间中有其他用户在上行视频数据时，您会收到 [onUserVideoAvailable()](http://doc.qcloudtrtc.com/group__TRTCCloudListener__android.html#ac1a0222f5b3e56176151eefe851deb05) 事件通知，不过此时 SDK 还不知道这些视频数据显示在哪里，因此并不会自动处理这些视频数据。您需要通过调用 [startRemoteView(userId, view)](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a57541db91ce032ada911ea6ea2be3b2c) 方法将远端用户的视频数据和显示 `view` 关联起来。
4. 您可以通过 [setRemoteViewFillMode()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#ab4197bc2efb62b471b49f926bab9352f) 指定视频画面的显示模式，其中 Fill 模式代表填充，此时画面可能会被等比放大和裁剪，但不会有黑边。Fit 模式代表适应，此时画面可能会等比缩小以完全显示其内容，可能会有黑边。
5. 您可以通过 [stopRemoteView(userId)](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a8f3e86bc219090d0e8f2d5c2fab4467a) 可以屏蔽某一个 userId 的视频数据，也可以通过 [stopAllRemoteView()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#addaac0786ac0bd6e73a5f35c038df127) 屏蔽所有远端用户的视频数据。之后 SDK 不再继续拉取这些远端用户的视频数据。

```
@Override
public void onUserVideoAvailable(String userId, boolean available) {
    TXCloudVideoView remoteView = remoteViewDic[userId];
    if (available) {
        trtcCloud.startRemoteView(userId, remoteView);
        trtcCloud.setRemoteViewFillMode(userId, TRTC_VIDEO_RENDER_MODE_FIT);
    } else {
        trtcCloud.stopRemoteView(userId);
    }
}
```

>? 如果您在收到 `onUserVideoAvailable()` 事件回调后没有立即调用 `startRemoteView()` 订阅视频流，SDK 并不会一直接收来自远端的视频数据，而是会在一段很短的时间后停止接收视频数据。

#### 手动订阅模式
您可以通过 [setDefaultStreamRecvMode()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a0b8d004665d5003ce1d9a48a9ab551b3) 接口将 SDK 指定为手动订阅模式。在手动订阅模式下，SDK 不会自动接收房间中其他用户的音视频数据，需要您手动通过 API 函数触发他们：

1. 在进房前调用 [setDefaultStreamRecvMode(false, false)](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a0b8d004665d5003ce1d9a48a9ab551b3) 接口将 SDK 设定为手动订阅模式。
2. 当房间中有其他用户在上行音频数据时，您会收到 [onUserAudioAvailable()](http://doc.qcloudtrtc.com/group__TRTCCloudListener__android.html#ac474bbf919f96c0cfda87c93890d871f) 事件通知。此时，您需要通过调用 [muteRemoteAudio(userId, false)](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a8d8b8edf120036d4049cc3639a1ce81f) 手动订阅该用户的音频数据，SDK 会在接收到该用户的音频数据后解码并播放之。
3. 当房间中有其他用户在上行视频数据时，您会收到 [onUserVideoAvailable()](http://doc.qcloudtrtc.com/group__TRTCCloudListener__android.html#ac1a0222f5b3e56176151eefe851deb05) 事件通知。此时，您需要通过调用 [startRemoteView(userId, remoteView)](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a57541db91ce032ada911ea6ea2be3b2c) 方法手动订阅该用户的视频数据，SDK 会在接收到该用户的视频数据后解码并播放之。

>! 接口 `setDefaultStreamRecvMode(false, false)` 需要在进房（`enterRoom`）前调用，否则无效。

<span id="step7"> </span>
### 步骤7：发布本地的音视频流
1. 调用 [startLocalAudio()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a9428ef48d67e19ba91272c9cf967e35e) 可以开启本地的麦克风采集，并将采集到的声音编码并发送出去。
2. 调用 [startLocalPreview()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a84098740a2e69e3d1f02735861614116) 可以开启本地的摄像头，并将采集到的画面编码并发送出去。
3. 调用 [setLocalViewFillMode()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#af36ab721c670e5871e5b21a41518b51d) 可以设定本地视频画面的显示模式，其中 Fill 模式代表填充，此时画面可能会被等比放大和裁剪，但不会有黑边。Fit 模式代表适应，此时画面可能会等比缩小以完全显示其内容，可能会有黑边。
4. 调用 [setVideoEncoderParam()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#ae047d96922cb1c19135433fa7908e6ce) 接口可以设定本地视频的编码参数，该参数决定了房间里其他用户观看您的画面时所感受到的[画面质量](https://cloud.tencent.com/document/product/647/32236)。

```java
//示例代码：发布本地的音视频流
trtcCloud.setLocalViewFillMode(TRTC_VIDEO_RENDER_MODE_FIT);
trtcCloud.startLocalPreview(mIsFrontCamera, mLocalView);
trtcCloud.startLocalAudio();
```

<span id="step8"> </span>
### 步骤8：正确的退出当前房间

调用 [exitRoom()](http://doc.qcloudtrtc.com/group__TRTCCloud__android.html#a41d16a97a9cb8f16ef92f5ef5bfebee1) 方法退出房间，由于 SDK 在退房时需要关闭和释放摄像头和麦克风等硬件设备，因此退房动作不是瞬间完成的，需要等待 [onExitRoom()](http://doc.qcloudtrtc.com/group__TRTCCloudListener__android.html#ad5ac26478033ea9c0339462c69f9c89e) 回调才算是真正退房结束。

```java
// 调用退房后请等待 onExitRoom 事件回调
trtcCloud.exitRoom()

@Override
public void onExitRoom(int reason) {
    Log.i(TAG, "onExitRoom: reason = " + reason);
}
```

>! 如果您在您的 App 中同时集成了多个音视频 SDK，请在收到 onExitRoom 回调后再启动其它音视频 SDK，否则可能会遭遇硬件占用问题。

