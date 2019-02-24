---
title: WebRTC p2p h5 api科普
date: 2018-07-12
description: WebRTC实现网页端到网页端互看的流程及demo
tags:
    - h5
    - WebRTC
author: jinqixiao
---
# WebRTC实现网页端到网页端互看的流程及demo

## - 什么是WebRTC

WebRTC是网页**实时通信**（Web Real-Time Communication）的缩写，可以让网页**无需安装插件**，进行**实时**语音对话或视频对话的技术，此例实现的是网页之间实时视频通信。

## 准备所需材料

2台有摄像头的电脑、高版本浏览器（这里用到是谷歌65）、信令服务器（可以用node实现，也可以用sip实现，作用是把页面的生成的数据发送给对端）、https服务器（webRTC访问本地设备需要https）

## WebRTC流程

![WebRTC流程图](../../../../img/h5_webrtc-h5api/pc流程图.png)

备注：图中Signal Server（黄色）为信令服务器，可以理解为传递数据用的后台，图中部分老规范方法已经弃用，需用新规范方法代替。

1.  A端和B端与信令服务建立连接，作为之后将A，B端消息交互的通道。

2.  A端调用创建WebRTC对象，传入一个参数，与打洞服务器有关。

``` js
let pc = new RTCPeerConnection(null);
```
3.  A端调用 **navigator.getUserMedia** 方法获取本地流stream，给video标签设置srcObject属性即可看到本地画面，并调用 **addTrack（图中AddStreams）** 方法将流添加至通道中。

``` js
//获取本地流
//第一个参数为分辨率限制，不需要则如下传值，第二个为成功回调，第三个为错误回调
//由于 navigator.getUserMedia 是异步方法，可修改为同步函数
let getNavMedia = () => {
  return new Promise((resolve, reject) => {
    navigator.getUserMedia({
      audio: true,
      video: true
    }, (stream) => {
      resolve(stream);
    }, (error) => {
      console.log(`navigator.getUserMedia error: ${error}`);
      resolve(error);
    });
  });
};

let mediaStream = await getNavMedia();
//设置video标签属性，显示视频
if (mediaStream.id) {
  myVideo.srcObject = mediaStream
}

// 在通道中添加流，图中add Streams过程
mediaStream.getTracks().forEach(
    function (track) {
        pc.addTrack(
            track,
            mediaStream
        );
    };
);
```

4.  A端调用 **createOffer** 生成offer（一个标准的sdp协议，大概包含音视频格式码流等信息），再调用 **setLocalDescription** 设置本地sdp（offer），并经过信令服务器发送至B端。

``` js
// createOffer返回一个promise，所以可以直接写成async
let offer = await pc.createOffer();

// 或者用.then来调用，成功回调返回一个offer参数
pc.createOffer().then(successBack, errorBack);

// setLocalDescription同createOffer，返回一个promise，只不过没有返回
pc.setLocalDescription(offer)
```

5.  B端接收到offer之后，调用 **setRemoteDescription** 设置远端sdp（offer），再调用 **createAnswer** 生产answer（与offer相对），调用 **setLocalDescription** 设置本地sdp（answer），并经过信令服务器发送至A端。

``` js
// setRemoteDescription, createAnswer皆返回promise
// 这里设置远端sdp完成才能生成对应answer，
// 否则会报错CreateAnswer can't be called before SetRemoteDescription.
await pc.setRemoteDescription(offer)

let answer = pc.createAnswer()

pc.setLocalDescription(answer)
```

6.  A端接收到answer后，调用 **setRemoteDescription** 设置远端sdp（answer）。
``` js
pc.setRemoteDescription(answer)
```

#### 以下为RTCPeerConnection事件，应该在第二步创建之后绑上

7.  第1-第6步过程中会生产icecandidate（浏览器打洞相关）数据，并且触发RTCPeerConnection对象的 **onicecandidate** 事件，这些icecandidate数据需要发送至对端，对端调用 **addIceCandidate** 添加。
``` js
// 产生icecandidate
pc.onicecandidate = (e) => {
  if (e.candidate) {
    console.log(e.candidate)
    ...// 经过信令服务器发送至对端
  }
}

// 接收icecandidate
pc.addIceCandidate(candidate)
```

8.  所有流程正常运行后，会触发pc的 **ontrack** 事件，并给出对端流，给video标签设置srcObject属性即可看到对端画面
``` js
pc.ontrack = (e) => {
  if (e.streams) {
    //设置video标签属性，显示视频    
    peerVideo.srcObject = e.streams[0]
  }
}
```

9.  **pc.onremovestream** 失去对端流事件

## 项目地址
[https://github.com/HowGraceU/WebRTC_p2p](https://github.com/HowGraceU/WebRTC_p2p) 

## 参考文献
WebRTC权威指南.pdf（已上传至项目中）