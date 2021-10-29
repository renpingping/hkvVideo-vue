<template>
  <div class="hkv-container">
    <el-row>
      <el-col :span="4" class="treeBox" 
        style="background: rgb(75,92,116);
        padding: 20px;
        border: 1px solid #325;">
        <channel />
      </el-col>
      <el-col :span="16" class="content">
        <div class="header">
          <el-radio-group v-model="tabPosition">
            <el-radio-button label="real"><i class="el-icon-video-camera-solid"></i>视频浏览</el-radio-button>
            <el-radio-button label="history"><i class="el-icon-video-camera-solid"></i> 视频回放</el-radio-button>
          </el-radio-group>
        </div>
        <div id="divPlugin" class="plugin"></div>
        <br/>
      </el-col>
      <el-col :span="4" class="ctrl">
        <cloudCtrl />
      </el-col>
    </el-row>


  </div>
</template>


<script>
  import cloudCtrl from './cloudCtrl'
  import channel from './channel'
  
  export default {
    components: {
      cloudCtrl,
      channel
    },
    data() {
      return {
        hkvInfo: {
          ip: '192.168.0.60',//海康威视摄像头/硬盘录像机的ip地址
          port: '80',//海康威视摄像头/硬盘录像机的端口
          username: 'admin',//海康威视摄像头/硬盘录像机的用户名
          password: 'pass@1234',//海康威视摄像头/硬盘录像机的密码
          channels: [2],//海康威视摄像头/硬盘录像机的通道
        },
        mySelectWnd: 0,//当前选中的窗口
        g_bPTZAuto: false,
        iProtocol: 1,
        loginLoading: false,
        startPlayLoading: false,
        iStreamType: 1,
        bZeroChannel: false,
        iRtspPort: 0,
        wndNum: 1, //展示窗口数
        wndNumList: [{
          value: 1,
          label: '1*1'
        },{
          value: 2,
          label: '2*2'
        },{
          value: 3,
          label: '3*3'
        },{
          value: 4,
          label: '4*4'
        }],
        tabPosition: 'real'
      }
    },
    created: function () {
      
        console.log(WebVideoCtrl)
    },
    mounted: function () {
      this.videoInitPlugin(); // 初始化video界面
    },

    destroyed: function () {
      this.clickStopRealPlay();
      this.onLogout();
    },
    methods: {
      videoInitPlugin: function () {
        var iRet = WebVideoCtrl.I_CheckPluginInstall();
        if (iRet === -1) {
          alert('您还未安装过插件，双击开发包目录里的WebComponentsKit.exe安装');
          return;
        }
        this.initPlugin()
      },
      initPlugin: function () {
        var that = this;

        WebVideoCtrl.I_InitPlugin(500, 300, {
          bWndFull: true,//是否支持单窗口双击全屏，默I_CheckPluginInstall
          iWndowType: 1,
          myCbSelWnd: function (xmlStr) { //自己新增的方法
            var jsonObj = that.$x2js.xml2js(xmlStr);
            that.mySelectWnd = jsonObj.RealPlayInfo.SelectWnd;
            var szInfo = "当前选择的窗口编号：" + that.mySelectWnd;
            console.log(szInfo);
          },
          cbInitPluginComplete: function () {
            WebVideoCtrl.I_InsertOBJECTPlugin("divPlugin");
            // 检查插件是否最新
            if (WebVideoCtrl.I_CheckPluginVersion() === -1) {
              alert("检测到新的插件版本，双击开发包目录里的WebComponentsKit.exe升级！");
              return;
            }
          }
        });
      },
      clickStopRealPlay: function () {
        var j = this.hkvInfo.channels.length > 4 ? 4 : this.hkvInfo.channels.length;
        for (var i = 0; i < j; i++) {
          setTimeout(this.stopRealPlay(i), 1000);
        }
      },
      stopRealPlay: function (iWndIndex) {
        var that = this;
        WebVideoCtrl.I_Stop({
          iWndIndex: iWndIndex,
          success: function () {
            that.$notify({
              title: '成功',
              message: '停止预览窗口' + iWndIndex + '成功',
              type: 'success'
            });
          },
          error: function () {
            that.$notify({
              title: '失败',
              message: '停止预览窗口' + iWndIndex + '失败',
              type: 'error'
            });
          }
        });
      },
      onLogout() {
        this.hkvInfo.channels = [];
        var szDeviceIdentify = this.hkvInfo.ip + "_" + this.hkvInfo.port;
        var iRet = WebVideoCtrl.I_Logout(szDeviceIdentify);
        if (0 == iRet) {
          this.$message({
            showClose: true,
            message: '退出成功',
            type: 'success'
          });
        } else {
          this.$message({
            showClose: true,
            message: '退出失败',
            type: 'error'
          });
        }
      },
      // onLogin() {
      //   var that = this;
      //   that.loginLoading = true;
      //   console.log(WebVideoCtrl)
      //   // 登录设备
      //   WebVideoCtrl.I_Login(that.hkvInfo.ip, that.iProtocol, that.hkvInfo.port, that.hkvInfo.username, that.hkvInfo.password, {
      //     async: false,
      //     success: function (xmlDoc) {
      //       // console.log('xmlDoc2', xmlDoc);//不能删除
      //       //TODO 获取通道信息
      //       that.loginLoading = false;
      //       that.getChannelInfo();
      //       that.getDevicePort(that.hkvInfo.ip + "_" + that.hkvInfo.port);

      //       that.$message({
      //         showClose: true,
      //         message: '登录成功',
      //         type: 'success'
      //       });
      //     },
      //     error: function () {
      //       that.loginLoading = false;
      //       that.$message({
      //         showClose: true,
      //         message: '登录失败',
      //         type: 'error'
      //       });
      //     }
      //   });
      // },
      // clickStartRealPlay() {
      //   // 开始预览
      //   var that = this;
      //   that.startPlayLoading = true;
      //   var szDeviceIdentify = that.hkvInfo.ip + "_" + that.hkvInfo.port;
      //   console.log('开始预览。。。。');
      //   var j = that.hkvInfo.channels.length > 4 ? 4 : that.hkvInfo.channels.length;
      //   console.log(that.hkvInfo, that.hkvInfo.channels, j);
      //   for (var i = 0; i < j; i++) {
      //     console.log(i);
      //     setTimeout(that.startRealPlay(szDeviceIdentify, i, that.hkvInfo.channels[i]), 500);
      //   }
      //   that.startPlayLoading = false;
      // },
      // getDevicePort: function (szDeviceIdentify) {
      //   var oPort = WebVideoCtrl.I_GetDevicePort(szDeviceIdentify);
      //   this.iRtspPort = oPort.iRtspPort;
      // },
      // startRealPlay(szDeviceIdentify, iWndIndex, iChannelID) {
      //   var that = this;
      //   console.log(szDeviceIdentify);
      //   WebVideoCtrl.I_StartRealPlay(szDeviceIdentify, {
      //     iRtspPort: that.iRtspPort,
      //     iWndIndex: iWndIndex,
      //     iStreamType: 1,
      //     iChannelID: iChannelID,
      //     bZeroChannel: that.bZeroChannel,
      //     success:  ()=> {
      //       that.$notify({
      //         title: '成功',
      //         message: '开始预览通道' + iChannelID + '成功',
      //         type: 'success'
      //       });
      //     },
      //     error: function (status, xmlDoc2) {
      //       console.log(xmlDoc2)//不能删除
      //       that.$notify({
      //         title: '失败',
      //         message: '开始预览通道' + iChannelID + '失败',
      //         type: 'error'
      //       });
      //       if (status === 403) {
      //         console.log("szInfo 设备不支持Websocket取流！");
      //       } else {
      //         console.log("开始预览失败 ", status, xmlDoc2);
      //       }
      //     }
      //   });
      // },
      // 获取通道，实际上可以根据自己的项目，获取数字通道，模拟通道，零通道中的一个或多个，不用全部获取（提高效率）
      // getChannelInfo: function () {
      //   var that = this;
      //   var szDeviceIdentify = this.hkvInfo.ip + "_" + this.hkvInfo.port;
      //   // debugger
      //   // 数字通道
      //   that.hkvInfo.channels = [];
      //   WebVideoCtrl.I_GetDigitalChannelInfo(szDeviceIdentify, {
      //       async: false,
      //       success: function (xmlStr) {
      //         console.log(that.$x2js)
              
      //         console.log('数字通道: ', xmlStr)
      //         let xml = that.XML2String(xmlStr)
      //         var jsonObj = that.$x2js.xml2js(xml)
      //         console.log("获取数字通道", jsonObj)
      //         if (jsonObj) {
      //           var list = jsonObj.InputProxyChannelStatusList.InputProxyChannelStatus;
      //           console.log(list);
      //           if (Array.isArray(list)) {
      //             for (var x = 0; x < list.length; x++) {
      //               that.hkvInfo.channels.push(list[x].id);
      //             }
      //           } else {
      //             that.hkvInfo.channels.push(list.id);
      //           }
      //         }
      //         console.log(that.hkvInfo.channels);
      //       },
      //       error: function (status, xmlDoc) {
      //         console.log("获取数字通道失败");
      //       }
      //     }
      //   );
      //   console.log(szDeviceIdentify);
      //   // TODO 模拟通道
      //   // 模拟通道
      //   WebVideoCtrl.I_GetAnalogChannelInfo(szDeviceIdentify, {
      //     async: false,
      //     success: function (xmlStr) {
      //       let xml = that.XML2String(xmlStr)
      //       var jsonObj = that.$x2js.xml2js(xml);
      //       console.log("模拟通道mysuccess",xml);
      //       var list = jsonObj.VideoInputChannelList;
      //       for (var x = 0; x < list.length; x++) {
      //         that.hkvInfo.channels.push(list[x].VideoInputChannel.id);
      //       }
      //       // var id = jsonObj.VideoInputChannelList.VideoInputChannel.id;
      //       that.hkvInfo.channels.push(id);
      //     },
      //     // success: function (xmlStr) {
      //     //   console.log("模拟通道success",xmlStr);
      //     // },
      //     error: function (status, xmlDoc) {
      //       // that.hkvInfo.channels.push(2);
      //       console.log("模拟通道error",xmlDoc);
      //     }
      //   });
      //   // TODO 零通道
      //   WebVideoCtrl.I_GetZeroChannelInfo(szDeviceIdentify, {
      //     async: false,
      //     success: function (xmlStr) {
      //       console.log('零通道 success: ', xmlStr)
      //       let xml = that.XML2String(xmlStr)
      //       var jsonObj = that.$x2js.xml2js(xml)
      //       console.log(jsonObj)
      //       var list = jsonObj.ZeroVideoChannelList;
      //       for (var x = 0; x < list.length; x++) {
      //         that.hkvInfo.channels.push(list[x].id);
      //       }
      //     },
      //     error: function (status, xmlDoc) {
      //         console.log("零通道error",xmlDoc);
      //     }
      //   });
      // },
      // mouseDownPTZControl: function (iPTZIndex) {
      //   console.log(iPTZIndex)
      //   var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.mySelectWnd);

      //   if (oWndInfo !== null) {
      //     if (iPTZIndex === 9 && this.g_bPTZAuto) {
      //       iPTZSpeed = 0;
      //     } else {
      //       this.g_bPTZAuto = false;
      //     }

      //     WebVideoCtrl.I_PTZControl(iPTZIndex, false, {
      //       iPTZSpeed: 4,
      //       mysuccess: function (xmlStr) {
      //         console.log("I_PTZControl", xmlStr);
      //         if (iPTZIndex === 9 && this.g_bPTZAuto) {
      //           console.log(oWndInfo.szDeviceIdentify + " 停止云台成功！mouseDown");
      //         } else {
      //           console.log(oWndInfo.szDeviceIdentify + " 开启云台成功！mouseDown");
      //         }
      //         if (iPTZIndex === 9) {
      //           this.g_bPTZAuto = !this.g_bPTZAuto;
      //         }
      //       },
      //       error: function (status, xmlDoc) {
      //         console.log(oWndInfo.szDeviceIdentify + " 开启云台失败！mouseDown", status, xmlDoc);
      //       }
      //     });
      //   }
      // },
      // mouseUpPTZControl: function () {
      //   var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.mySelectWnd);
      //   if (oWndInfo !== null) {
      //     WebVideoCtrl.I_PTZControl(1, true, {
      //       mysuccess: function (xmlStr) {
      //         console.log(oWndInfo.szDeviceIdentify + " 停止云台成功！mouseUp", xmlStr)
      //       },
      //       error: function (status, xmlDoc) {
      //         console.log(oWndInfo.szDeviceIdentify + " 停止云台失败！mouseUp", status, xmlDoc);
      //       }
      //     });
      //   }
      // },
      // 窗口分割数
      // changeWndNum(iType) {
      //   console.log(iType);
      //   iType = parseInt(iType, 10);
      //   WebVideoCtrl.I_ChangeWndNum(iType);
      // },
      // convert XML object to string  兼容ie
    //   XML2String: function (xmlobject) {
    //     // for IE
    //     if (window.ActiveXObject) {
    // 　　　　return xmlobject.xml;
    //     }
    //     // for other browsers
    //     else {       
    //       return (new XMLSerializer()).serializeToString(xmlobject);
    //     }
    //   },
    //   //convert string to xml object 兼容ie
    //   String2XML(xmlstring) {
    //     // for IE
    //     if (window.ActiveXObject) {
    //       var xmlobject = new ActiveXObject("Microsoft.XMLDOM");
    //       xmlobject.async = "false";
    //       xmlobject.loadXML(xmlstring);
    //       return xmlobject;
    //     }
    //     // for other browsers
    //     else {
    //       var parser = new DOMParser();
    //       var xmlobject = parser.parseFromString(xmlstring, "text/xml");
    //       return xmlobject;
    //     }
    //   }
    }
  }
</script>

<style scoped>
  .plugin {
    width: 1000px;
    height: 600px;
  }

  .my-tag {
    margin-left: 3px;
  }

  .my-group-btn {
    margin-top: 5px;
  }
  .hkv-container {
    background: rgb(10, 27, 55);
  }
  .treeBox {
    width: 270px;
  }
  .content {
    margin: auto 10px;
    width: 65%;
    max-width: 1106px;
    /* background: rosybrown; */
  }
  .ctrl {
    width: 16.8%;
    max-width: 270px;
    color: azure;
    /* background: darkcyan; */
  }
  .header {
    width: 100%;
    margin-bottom: 5px;
    padding: 4px;
    border: 2px solid rgb(46 91 116);
    background: #223d54;
    font-size: 16px;
    color: #efefef;
  }
  .header .el-radio-group .el-radio-button .el-radio-button__inner {
    background: transparent!important;;
    border: none;
    font-size: 16px;
  }
</style>
