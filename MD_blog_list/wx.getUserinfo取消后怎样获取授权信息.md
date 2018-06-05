wx.getUserinfo从2018年4月30号开始不在支持，然后再官方文档里没有找到一个完整的demo去完成这一过程；经过不断探索，写出一个相对完善的demo;  
效果如下：  
![demo效果图](https://github.com/liyiyy/MarkdownPhotos/blob/master/images/01/wxmini-01.gif)


逻辑分析：
进入小程序时，判断是否存在openid，若有则直接判断是否已经投票，已经投票则直接去结果页；   若没有，则通过wx.login获取并setStorage；    然后判断是否获取用户授权，获取授权就获取用户信息，并判断投票到结果页；   
未授权就打开授权页面进行授权，拒绝就直接返回，知道用户授权获取到用户信息在提交；
```
graph TD
A[init] -->B{have openid}
B -->|no| C[wx:login & setStorage openid userid]
B -->|have| F[11]
C --> D{is bindgetuserinfo}
D -->|yes| F[e.detail.userinfo]
D -->|no| E{!e.detail.userinfo}
F --> G{isVote}
G -->|yes| H[go result page]
G -->|no| I[suresub]
E -->|yes| F
E -->|no| K[opensetting]
K -->|yes or no| D
```
==煞费苦心的写了这么久的流程图，居然显示不出来，心塞塞。流程图显示如下==
![流程图](https://github.com/liyiyy/MarkdownPhotos/blob/master/images/01/02.png)

代码如下：


###### app.js
```
onLaunch: function () {
    var that = this;
    var openid = wx.getStorageSync('openid');
    if (!openid) {
      wx.login({
        success: function (res) {
          console.log("登录成功.msg:", res.errMsg, ",code:", res.code);

          wx.request({
            url: config.oppenUrl,  // 在demo/utils/require.js 里面配置请求url
            data: {
              code: res.code
            },
            header: {
              'content-type': 'application/json' // 默认值
            },
            method: 'GET',
            success: function (res) {
              // console.log(res.data)
              if (res.data.message == "SUCCESS") {
                // 用户信息存入Storage
                wx.setStorageSync('openid', res.data.data.openid);
                if (res.data.data.user_id > 0) {
                  wx.setStorageSync('userid', res.data.data.user_id)
                }
              }

            },
            fail: function (e) {
              console.log('request fail')
            }
          })
        }
      });
    }
  },

```
###### index.wxml
```
<button type="primary" style='width:90px;' wx:if="{{canIUse}}" open-type="getUserInfo" bindgetuserinfo="bindGetUserInfo">提交</button>
```
 在更新的版本中；必须加 open-type=""  ；值有如下：  
 值 | 	说明 | 	最低版本
---|---|---
contact	 |  打开客服会话 |	1.1.0
share	|触发用户转发，使用前建议先阅读使用指引	|1.2.0
getUserInfo	|获取用户信息，可以从bindgetuserinfo回调中获取到用户信息|	1.3.0
getPhoneNumber|	获取用户手机号，可以从bindgetphonenumber回调中获取到用户信息，具体说明|	1.2.0
launchApp|	打开APP，可以通过app-parameter属性设定向APP传的参数具体说明	|1.9.5
openSetting |	打开授权设置页|	2.0.7
==煞费苦心的写的表格，居然显示不出来，心塞塞。显示如下==
![流程图](https://github.com/liyiyy/MarkdownPhotos/blob/master/images/01/03.png)
###### index.js
```
//  获取用户信息
  bindGetUserInfo: function (e) {
    if (e.detail.userInfo) {
      this.getSet(e.detail.userInfo)
    } else {
      this.openSet()
    }
  },
  
  //  授权成功后的调用
  getSet(userinfo) {
    var userid = wx.getStorageSync('userid')
    if (userid) {
      this.suresub()
    } else {
      var data = {
        openid: wx.getStorageSync('openid'),
        userinfo: userinfo
      }
      app.request(config.addUser, data, 'GET').then(res => {
        if (res.user_id > 0) {
          wx.setStorageSync('userid', res.user_id)
          this.suresub()
        }
      }).catch((err) => {
        wx.showToast({
          title: err,
          icon: 'none'
        })
      })
    }
  },
  
  //  拒绝授权后发起授权、
  openSet() {
    var that = this;
    wx.showModal({
      title: '提示',
      content: '您拒绝授权，不能发起投票，点击重新获取授权',
      showCancel: true,
      cancelText: '取消授权',
      confirmText: '去授权',
      success: function (res) {
        if (res.confirm) {
          wx.openSetting({
            success: function (res) {
              if (res.authSetting["scope.userInfo"]) {
                that.getSet("");
              } else {
                that.openSet();
              }
            }
          })
        }
      }
    })
  },
```




























