<template>
  <div class="channel-container">
    <el-tree
      :load="loadNode"
      lazy
      :data="data"
      :props="defaultProps"
      :accordion="true"
      @node-click="handleNodeClick"
      style="min-height: 500px"
    ></el-tree>
    <!-- <div>
      <el-form :inline="true" :model="hkvInfo" class="demo-form-inline">
        <el-row>
          <el-col>
            <el-form-item label="ip">
              <el-input v-model="hkvInfo.ip" placeholder="ip"></el-input>
            </el-form-item>
            <el-form-item label="port">
              <el-input v-model="hkvInfo.port" placeholder="port"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col>
            <div>
              <el-form-item label="username">
                <el-input
                  v-model="hkvInfo.username"
                  placeholder="username"
                ></el-input>
              </el-form-item>
              <el-form-item label="password">
                <el-input
                  v-model="hkvInfo.password"
                  placeholder="password"
                ></el-input>
              </el-form-item>
            </div>
          </el-col>
        </el-row>
        <el-row>
          <el-col>
            <el-form-item>
              <el-button type="primary" :loading="loginLoading" @click="onLogin"
                >登录</el-button
              >
              <el-button
                type="primary"
                :loading="startPlayLoading"
                @click="clickStartRealPlay"
                >开始预览</el-button
              >
              <el-button type="primary" @click="clickStopRealPlay"
                >停止预览</el-button
              >
              <el-button type="primary" @click="onLogout">退出</el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </div> -->
    <div>
      <label>数字通道列表: </label>
      <el-tag
        type="success"
        class="my-tag"
        v-for="(item, index) in hkvInfo.channels"
        :key="index"
        >{{ item }}</el-tag
      >
    </div>
    <div>
      <label>窗口分割：</label>
      <el-select v-model="wndNum" placeholder="请选择" @change="changeWndNum">
        <el-option
          v-for="item in wndNumList"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        >
        </el-option>
      </el-select>
    </div>
  </div>
</template>


<script>
export default {
  components: {},
  data() {
    return {
      hkvInfo: {
        ip: "192.168.0.60", //海康威视摄像头/硬盘录像机的ip地址
        port: "80", //海康威视摄像头/硬盘录像机的端口
        username: "admin", //海康威视摄像头/硬盘录像机的用户名
        password: "pass@1234", //海康威视摄像头/硬盘录像机的密码
        channels: [2], //海康威视摄像头/硬盘录像机的通道
        channelsName: [], //海康威视摄像头/硬盘录像机的通道
      },
      mySelectWnd: 0, //当前选中的窗口
      g_bPTZAuto: false,
      iProtocol: 1,
      loginLoading: false,
      startPlayLoading: false,
      iStreamType: 1,
      bZeroChannel: false,
      iRtspPort: 0,
      wndNum: 1, //展示窗口数
      wndNumList: [
        {
          value: 1,
          label: "1*1",
        },
        {
          value: 2,
          label: "2*2",
        },
        {
          value: 3,
          label: "3*3",
        },
        {
          value: 4,
          label: "4*4",
        },
      ],
      // 树形数据
      data: [
        {
          label: "192.168.0.60",
          meta:{
            ip: "192.168.0.60",
            port: '80',
            username: 'admin',
            password: 'pass@1234',
            level: 0
          },
        },
        {
          label: "192.168.2.61",
          meta:{
            ip: "192.168.2.61",
            port: '80',
            username: 'admin',
            password: 'pass@1234',
            level: 0
          },
        },
        {
          label: "192.168.2.62",
          meta:{
            ip: "192.168.2.62",
            port: '80',
            username: 'admin',
            password: 'pass@1234',
            level: 0
          },
        },
      ],
      defaultProps: {
        children: "children",
        label: "label",
        isLeaf: 'leaf'
      },
    };
  },
  created: function () {
    console.log(WebVideoCtrl);
  },
  mounted: function () {
    // this.videoInitPlugin(); // 初始化video界面
  },

  destroyed: function () {
    this.clickStopRealPlay();
    this.onLogout();
  },
  methods: {
    handleNodeClick(data, node, tree) {
      console.log(data, node, tree);
      if (data.meta.level === 0) {
        this.hkvInfo = data.meta
        this.onLogin()
      } else {
        // 点击通道 开始预览
        let parent = node.parent.data
        this.hkvInfo = parent.meta
        console.log(data);
        this.hkvInfo.channels = [data.meta.id]
        this.clickStartRealPlay()
      }
    },
    async loadNode(node, resolve) {
      if (node.level === 0) {
        return resolve(this.data);
      }
      if (node.level > 1) return resolve([]);
      console.log('>>>>', this.hkvInfo.channels);
      setTimeout(() => {
        console.log('<<<<', this.hkvInfo.channels);
        if (this.hkvInfo.channels) {
          const data = this.hkvInfo.channels.map((item, index) => {
            return {
              label: this.hkvInfo.channelsName[index], 
              meta: {
                id: item,
                level: 1,
              },
              leaf: true
            }
          })
          resolve(data);
        } else {
          return resolve([]);
        }
      }, 500);
    },
    onLogin() {
      var that = this;
      that.loginLoading = true;
      console.log(WebVideoCtrl);
      // 登录设备
      WebVideoCtrl.I_Login(
        that.hkvInfo.ip,
        that.iProtocol,
        that.hkvInfo.port,
        that.hkvInfo.username,
        that.hkvInfo.password,
        {
          async: false,
          success: function (xmlDoc) {
            // console.log('xmlDoc2', xmlDoc);//不能删除
            //TODO 获取通道信息
            that.loginLoading = false;
            that.getChannelInfo();
            that.getDevicePort(that.hkvInfo.ip + "_" + that.hkvInfo.port);
            console.log('>>> 登录成功 <<<')
          },
          error: function () {
            that.loginLoading = false;
            console.log('>>> 登录失败 <<<')
          },
        }
      );
    },
    onLogout() {
      this.hkvInfo.channels = [];
      var szDeviceIdentify = this.hkvInfo.ip + "_" + this.hkvInfo.port;
      var iRet = WebVideoCtrl.I_Logout(szDeviceIdentify);
      if (0 == iRet) {
        this.$message({
          showClose: true,
          message: "退出成功",
          type: "success",
        });
      } else {
        this.$message({
          showClose: true,
          message: "退出失败",
          type: "error",
        });
      }
    },
    clickStartRealPlay() {
      // 开始预览
      var that = this;
      that.startPlayLoading = true;
      var szDeviceIdentify = that.hkvInfo.ip + "_" + that.hkvInfo.port;
      console.log("开始预览。。。。");
      var j =
        that.hkvInfo.channels.length > this.wndNum ? this.wndNum : that.hkvInfo.channels.length;
      console.log(that.hkvInfo, that.hkvInfo.channels, j);
      for (var i = 0; i < j; i++) {
        console.log(i);
        setTimeout(
          that.startRealPlay(szDeviceIdentify, i, that.hkvInfo.channels[i]),
          500
        );
      }
      that.startPlayLoading = false;
    },
    // videoInitPlugin: function () {
    //   var iRet = WebVideoCtrl.I_CheckPluginInstall();
    //   if (iRet === -1) {
    //     alert("您还未安装过插件，双击开发包目录里的WebComponentsKit.exe安装");
    //     return;
    //   }
    //   this.initPlugin();
    // },
    // initPlugin: function () {
    //   var that = this;

    //   WebVideoCtrl.I_InitPlugin(500, 300, {
    //     bWndFull: true, //是否支持单窗口双击全屏，默I_CheckPluginInstall
    //     iWndowType: 1,
    //     myCbSelWnd: function (xmlStr) {
    //       //自己新增的方法
    //       var jsonObj = that.$x2js.xml2js(xmlStr);
    //       that.mySelectWnd = jsonObj.RealPlayInfo.SelectWnd;
    //       var szInfo = "当前选择的窗口编号：" + that.mySelectWnd;
    //       console.log(szInfo);
    //     },
    //     cbInitPluginComplete: function () {
    //       WebVideoCtrl.I_InsertOBJECTPlugin("divPlugin");
    //       // 检查插件是否最新
    //       if (WebVideoCtrl.I_CheckPluginVersion() === -1) {
    //         alert(
    //           "检测到新的插件版本，双击开发包目录里的WebComponentsKit.exe升级！"
    //         );
    //         return;
    //       }
    //     },
    //   });
    // },
    getDevicePort: function (szDeviceIdentify) {
      var oPort = WebVideoCtrl.I_GetDevicePort(szDeviceIdentify);
      this.iRtspPort = oPort.iRtspPort;
    },
    startRealPlay(szDeviceIdentify, iWndIndex, iChannelID) {
      var that = this;
      console.log(szDeviceIdentify);
      WebVideoCtrl.I_StartRealPlay(szDeviceIdentify, {
        iRtspPort: that.iRtspPort,
        iWndIndex: iWndIndex,
        iStreamType: 1,
        iChannelID: iChannelID,
        bZeroChannel: that.bZeroChannel,
        success: () => {
          that.$notify({
            title: "成功",
            message: "开始预览通道" + iChannelID + "成功",
            type: "success",
          });
        },
        error: function (status, xmlDoc2) {
          console.log(xmlDoc2); //不能删除
          that.$notify({
            title: "失败",
            message: "开始预览通道" + iChannelID + "失败",
            type: "error",
          });
          if (status === 403) {
            console.log("szInfo 设备不支持Websocket取流！");
          } else {
            console.log("开始预览失败 ", status, xmlDoc2);
          }
        },
      });
    },
    clickStopRealPlay: function () {
      var j =
        this.hkvInfo.channels.length > 4 ? 4 : this.hkvInfo.channels.length;
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
            title: "成功",
            message: "停止预览窗口" + iWndIndex + "成功",
            type: "success",
          });
        },
        error: function () {
          that.$notify({
            title: "失败",
            message: "停止预览窗口" + iWndIndex + "失败",
            type: "error",
          });
        },
      });
    },
    // 获取通道，实际上可以根据自己的项目，获取数字通道，模拟通道，零通道中的一个或多个，不用全部获取（提高效率）
    getChannelInfo() {
      var that = this;
      var szDeviceIdentify = this.hkvInfo.ip + "_" + this.hkvInfo.port;
      // debugger
      // 数字通道
      this.hkvInfo.channels = [];
      this.hkvInfo.channelsName = [];
      WebVideoCtrl.I_GetDigitalChannelInfo(szDeviceIdentify, {
        async: false,
        success: function (xmlStr) {
          console.log(that.$x2js);

          console.log("数字通道: ", xmlStr);
          let xml = that.XML2String(xmlStr);
          var jsonObj = that.$x2js.xml2js(xml);
          console.log("获取数字通道", jsonObj);
          if (jsonObj) {
            var list =
              jsonObj.InputProxyChannelStatusList.InputProxyChannelStatus;
            console.log(list);
            if (Array.isArray(list)) {
              for (var x = 0; x < list.length; x++) {
                that.hkvInfo.channels.push(list[x].id);
                that.hkvInfo.channelsName.push(list[x].sourceInputPortDescriptor.name);
              }
            } else {
              that.hkvInfo.channels.push(list.id);
              that.hkvInfo.channelsName.push(list.sourceInputPortDescriptor.name);
            }
          }
          console.log(that.hkvInfo.channelsName);
        },
        error: function (status, xmlDoc) {
          console.log("获取数字通道失败");
        },
      });
      console.log(szDeviceIdentify);
      // TODO 模拟通道
      // 模拟通道
      WebVideoCtrl.I_GetAnalogChannelInfo(szDeviceIdentify, {
        async: false,
        success: function (xmlStr) {
          let xml = that.XML2String(xmlStr);
          var jsonObj = that.$x2js.xml2js(xml);
          console.log("模拟通道mysuccess", xml);
          var list = jsonObj.VideoInputChannelList;
          for (var x = 0; x < list.length; x++) {
            that.hkvInfo.channels.push(list[x].VideoInputChannel.id);
            that.hkvInfo.channelsName.push(list[x].VideoInputChannel.name);
          }
          // var id = jsonObj.VideoInputChannelList.VideoInputChannel.id;
          // that.hkvInfo.channels.push(id);
        },
        error: function (status, xmlDoc) {
          // that.hkvInfo.channels.push(2);
          console.log("模拟通道error", xmlDoc);
        },
      });
      // TODO 零通道
      WebVideoCtrl.I_GetZeroChannelInfo(szDeviceIdentify, {
        async: false,
        success: function (xmlStr) {
          console.log("零通道 success: ", xmlStr);
          let xml = that.XML2String(xmlStr);
          var jsonObj = that.$x2js.xml2js(xml);
          console.log(jsonObj);
          var list = jsonObj.ZeroVideoChannelList;
          for (var x = 0; x < list.length; x++) {
            that.hkvInfo.channels.push(list[x].id);
            that.hkvInfo.channelsName.push(list[x].id);
          }
        },
        error: function (status, xmlDoc) {
          console.log("零通道error", xmlDoc);
        },
      });
      console.log(this.hkvInfo.channelsName);
    },
    mouseDownPTZControl: function (iPTZIndex) {
      console.log(iPTZIndex);
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.mySelectWnd);

      if (oWndInfo !== null) {
        if (iPTZIndex === 9 && this.g_bPTZAuto) {
          iPTZSpeed = 0;
        } else {
          this.g_bPTZAuto = false;
        }

        WebVideoCtrl.I_PTZControl(iPTZIndex, false, {
          iPTZSpeed: 4,
          mysuccess: function (xmlStr) {
            console.log("I_PTZControl", xmlStr);
            if (iPTZIndex === 9 && this.g_bPTZAuto) {
              console.log(
                oWndInfo.szDeviceIdentify + " 停止云台成功！mouseDown"
              );
            } else {
              console.log(
                oWndInfo.szDeviceIdentify + " 开启云台成功！mouseDown"
              );
            }
            if (iPTZIndex === 9) {
              this.g_bPTZAuto = !this.g_bPTZAuto;
            }
          },
          error: function (status, xmlDoc) {
            console.log(
              oWndInfo.szDeviceIdentify + " 开启云台失败！mouseDown",
              status,
              xmlDoc
            );
          },
        });
      }
    },
    mouseUpPTZControl: function () {
      var oWndInfo = WebVideoCtrl.I_GetWindowStatus(this.mySelectWnd);
      if (oWndInfo !== null) {
        WebVideoCtrl.I_PTZControl(1, true, {
          mysuccess: function (xmlStr) {
            console.log(
              oWndInfo.szDeviceIdentify + " 停止云台成功！mouseUp",
              xmlStr
            );
          },
          error: function (status, xmlDoc) {
            console.log(
              oWndInfo.szDeviceIdentify + " 停止云台失败！mouseUp",
              status,
              xmlDoc
            );
          },
        });
      }
    },
    // 窗口分割数
    changeWndNum(iType) {
      console.log(iType);
      iType = parseInt(iType, 10);
      WebVideoCtrl.I_ChangeWndNum(iType);
    },
    // convert XML object to string  兼容ie
    XML2String: function (xmlobject) {
      // for IE
      if (window.ActiveXObject) {
        return xmlobject.xml;
      }
      // for other browsers
      else {
        return new XMLSerializer().serializeToString(xmlobject);
      }
    },
    //convert string to xml object 兼容ie
    String2XML(xmlstring) {
      // for IE
      if (window.ActiveXObject) {
        var xmlobject = new ActiveXObject("Microsoft.XMLDOM");
        xmlobject.async = "false";
        xmlobject.loadXML(xmlstring);
        return xmlobject;
      }
      // for other browsers
      else {
        var parser = new DOMParser();
        var xmlobject = parser.parseFromString(xmlstring, "text/xml");
        return xmlobject;
      }
    },
  },
};
</script>

<style scoped>
.channel-container {
  color: aliceblue;
}
.el-tree {
  color: #409eff;
  background: transparent;
}
</style>
