wx.getUserinfo从2018年4月30号开始不在支持，然后再官方文档里没有找到一个完整的demo去完成这一过程；经过不断探索，写出一个相对完善的demo;  
效果如下：  
![demo效果图](https://github.com/liyiyy/MarkdownPhotos/blob/master/images/01/wxmini-01.gif)


逻辑分析：
进入小程序时，判断是否存在openid，若有则直接判断是否已经投票，已经投票则直接去结果页；   若没有，则通过wx.login获取并setStorage；    然后判断是否获取用户授权，获取授权就获取用户信息，并判断投票到结果页；   
未授权就打开授权页面进行授权，拒绝就直接返回，知道用户授权获取到用户信息在提交；

![流程图](https://github.com/liyiyy/MarkdownPhotos/blob/master/images/01/02.png)

代码如下：


###### app.js
```
onLaunch: function () {
    this.login()
  },
  login(){
    var that = this;
    // 小程序调用wx.login() 获取 临时登录凭证code ，并回传到开发者服务器。
    // 开发者服务器以code换取 用户唯一标识openid 和 会话密钥session_key。
    var openid = wx.getStorageSync('openid');
    if (!openid) {
      wx.login({
        success: function (res) {
          console.log("登录成功.msg:", res.errMsg, ",code:", res.code);

          wx.request({
            url: config.oppenUrl,
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
  ...

```
###### index.wxml
```
<button type="primary" style='width:90px;' wx:if="{{canIUse}}" open-type="getUserInfo" bindgetuserinfo="bindGetUserInfo">提交</button>
```
 在更新的版本中；必须加 open-type=""  ；值有如下： 

![open-type值表](https://github.com/liyiyy/MarkdownPhotos/blob/master/images/01/03.png)
###### index.js
```
//  获取用户信息
  bindGetUserInfo: function (e) {
    if (e.detail.userInfo) {     
      // 授权后的调用
      var userid = wx.getStorageSync('userid')
      if (userid) {
        this.suresub()
      } else {
        var data = {
          openid: wx.getStorageSync('openid'),
          userinfo: e.detail.userInfo
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
    } else {
      this.openSet()
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
              console.log(res);
              if (res.authSetting["scope.userInfo"]) {
                wx.showToast({
                  title: '用户信息获取成功，请重新提交',
                  icon:'none'
                })
                app.login()

              } else {
                that.openSet();
              }
            }
          })
        }else{
          wx.showToast({
            title: '用户信息获取失败，请重新获取',
            icon: 'none'
          })
        }
      }
    })
  },
```




























