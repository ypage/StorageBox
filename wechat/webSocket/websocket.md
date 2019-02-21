# WeChat

标签（空格分隔）：Mini-Program

---

###  ---目录---
[TOC]

什么是WebSocket？

    WebSocket是一种自然的全双工、双向、单套接字连接。使用WebSocket，你的HTTP 请求变成打开WebSocket 连接（WebSocket 或者WebSocket over TLS（TransportLayer Security，传输层安全性，原称“SSL”））的单一请求，并且重用从客户端到服务器以及服务器到客户端的同一连接。
WebSocket 构造函数

    WebSocket 协议定义了两种URL 方案（URL scheme）—ws 和wss，分别用于客户端和服务器之间的非加密与加密流量。ws（WebSocket） 方案与HTTP URI 方案类似。wss（WebSocketSecure，WebSocket 安全）URI 方案表示使用传输层安全性（TLS，也叫SSL）的WebSocket 连接，使用HTTPS 采用的安全机制来保证HTTP 连接的安全。

### 1. webSocket连接
```
var BaseUrl = 'wss://weace.cn/ypage-websocket/'
```

### 2. 初始化webSocket连接
```
initWebSocket() {
    var o = this;
    wx.onSocketMessage((res) => {
        let dataList = JSON.parse(res.data),
        chatList = o.data.chatList
        if (dataList.type == 'history') {
            let dataJs = JSON.parse(dataList.data)
            o.setData({
                chatList: dataJs
            })
        }
        if ((dataList.type != 'states') && (dataList.type != 'history')) {
            let i = 0;
            chatList.forEach(v => {
                if (v.fromId == dataList.fromId) {
                    chatList.splice(i, 1);
                }
                i++;
            });
            chatList.unshift(dataList)
            o.setData({
                chatList: chatList
            })
        }
    })
    wx.onSocketOpen(() => {
        console.log('websocket重连打开!')
    })
    wx.onSocketError((res) => {
        console.log('websocket重连失败!')
        o.reConnect()
    })
    wx.onSocketClose((res) => {
        console.log('websocket关闭!')
        o.reConnect()
    })
  }
```

### 3. 离开页面close连接
```
goPage(e) {
    var o = this;
    let chatList = o.data.chatList
    if (chatList.length > 0) {
        chatList.forEach(v => {
            if (v.fromId == e.currentTarget.dataset.id) {
            v.states = 1
            }
        });
    } else if (chatList.fromId == e.currentTarget.dataset.id) {
        chatList.states = 1
    }
    o.repHistory()
    wx.closeSocket();
    wx.navigateTo({
        url: './message/index?name=' + e.currentTarget.dataset.name + "&id=" + e.currentTarget.dataset.id
    })
}
```



