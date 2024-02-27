# web2app.co

## 安卓客户端集成
将libs下的jar文件放入安卓工程，kotlin代码如下
```kotlin
val channelInfo: ChannelInfo? = WalleChannelReader.getChannelInfo(activity)
android.util.Log.i("channel_info", "channel_info:${channelInfo?.channel}    ${channelInfo?.extraInfo}")
val map = mutableMapOf<String, String>();
if (channelInfo != null) {
    map["channel"] = channelInfo.channel
    if (channelInfo.extraInfo != null && channelInfo.extraInfo["adinfo"] != null){
        map["adinfo"] = channelInfo.extraInfo["adinfo"]!!
    }
}
```

## 归因事件回传
```js
const url = `https://api.web2app.co/conversion?event=${event_name}&data=${JSON.stringify(event_data)}&url=${adinfo}`;
fetch(url)
```
将取到的adinfo原样拼接到url参数中，data为可选参数，内容参考google/facebook的文档，以购买为例，内容为{ value: 1.99, currency: 'USD' }
返回值为：
```json
{
    "ok": true,     // true为成功，false为失败
    "reason": "ok", // 失败时会给出原因
    "message": "Endpoint timeout." // 如果api gateway超时，会没有以上两条数据而只有这一条。
}
```

## 落地页
落地页根据google/facebook提供的代码集成，确保firebase(google)或者fbq(facebook)全局变量可用。
将apk传给我们后，便可以通过
```js
window.location.href = `https://ad.web2app.co/download?apk=${apk_name}&url=${encodeURIComponent(window.location.href)}`
```
来下载集成对应adinfo的包，安装后便可根据adinfo做广告归因了

## 新版firebase的处理办法
如果落地页是按照新版firebase的module方式集成，需要做如下处理
```js
<script type="module">
import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.4.0/firebase-app.js'
import { getAnalytics, logEvent } from 'https://www.gstatic.com/firebasejs/10.4.0/firebase-analytics.js'

const firebaseConfig = {...};

const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app); 
window.firebaseLogEvent = function (event, data) { logEvent(analytics, event, data) }
</script>
```