# WeChat

��ǩ���ո�ָ�����Mini-Program

---

###  ---Ŀ¼---
[TOC]

ʲô��WebSocket��

    WebSocket��һ����Ȼ��ȫ˫����˫�򡢵��׽������ӡ�ʹ��WebSocket�����HTTP �����ɴ�WebSocket ���ӣ�WebSocket ����WebSocket over TLS��TransportLayer Security������㰲ȫ�ԣ�ԭ�ơ�SSL�������ĵ�һ���󣬲������ôӿͻ��˵��������Լ����������ͻ��˵�ͬһ���ӡ�
WebSocket ���캯��

    WebSocket Э�鶨��������URL ������URL scheme����ws ��wss���ֱ����ڿͻ��˺ͷ�����֮��ķǼ��������������ws��WebSocket�� ������HTTP URI �������ơ�wss��WebSocketSecure��WebSocket ��ȫ��URI ������ʾʹ�ô���㰲ȫ�ԣ�TLS��Ҳ��SSL����WebSocket ���ӣ�ʹ��HTTPS ���õİ�ȫ��������֤HTTP ���ӵİ�ȫ��

### 1. webSocket����
```
var BaseUrl = 'wss://weace.cn/ypage-websocket/'
```

### 2. ��ʼ��webSocket����
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
        console.log('websocket������!')
    })
    wx.onSocketError((res) => {
        console.log('websocket����ʧ��!')
        o.reConnect()
    })
    wx.onSocketClose((res) => {
        console.log('websocket�ر�!')
        o.reConnect()
    })
  }
```

### 3. �뿪ҳ��close����
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



