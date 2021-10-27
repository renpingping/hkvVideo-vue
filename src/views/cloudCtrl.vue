<template>
  <div class="cloud-container">
    <div class="header"><i class="el-icon-s-help"></i> 云台操作</div>
    <div class="warpper">
      <div class="content">
        <div class="el-button-group">
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(5)"
            @mouseup="mouseUpPTZControl"
          >
            <!-- <i class="el-icon-caret-top" style="transform: rotate(-45deg);"></i> -->
          </div>
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(1)"
            @mouseup="mouseUpPTZControl"
          >
            <i class="el-icon-caret-top"></i>
          </div>
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(7)"
            @mouseup="mouseUpPTZControl"
          >
            <!-- <i class="el-icon-caret-top" style="transform: rotate(45deg);"></i> -->
          </div>
        </div>
        <br />
        <div class="el-button-group">
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(3)"
            @mouseup="mouseUpPTZControl"
          >
            <i class="el-icon-caret-left"></i>
          </div>
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(9)"
            @mouseup="mouseUpPTZControl"
          >
            <i class="el-icon-refresh"></i>
          </div>
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(4)"
            @mouseup="mouseUpPTZControl"
          >
            <i class="el-icon-caret-right"></i>
          </div>
        </div>
        <br />
        <div class="el-button-group">
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(6)"
            @mouseup="mouseUpPTZControl"
          >
            <!-- <i class="el-icon-caret-bottom" style="transform: rotate(45deg);"></i> -->
          </div>
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(2)"
            @mouseup="mouseUpPTZControl"
          >
            <i class="el-icon-caret-bottom"></i>
          </div>
          <div
            class="el-button primary"
            @mousedown="mouseDownPTZControl(8)"
            @mouseup="mouseUpPTZControl"
          >
            <!-- <i class="el-icon-caret-bottom" style="transform: rotate(-45deg);"></i> -->
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      mySelectWnd: 0, //当前选中的窗口
      g_bPTZAuto: false,
    };
  },
  created: function () {
    console.log(WebVideoCtrl);
  },
  mounted: function () {},

  destroyed: function () {
    this.clickStopRealPlay();
    this.onLogout();
  },
  methods: {
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
.cloud-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
}
.header {
  width: 242px;
  margin-bottom: 5px;
  padding: 14px;
  border: 2px solid rgb(46 91 116);
  background: #223d54;
  font-size: 16px;
  color: #efefef;
}
.warpper {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  flex-direction: column;
  flex-wrap: nowrap;
  align-content: center;
  width: 270px;
  height: 380px;
  border: 2px solid rgb(46 91 116);
  background: #223d54;
}
.content {
  position: relative;
  margin-top: 20px;
  border-radius: 50%;
  background: rgb(212, 226, 245);
  width: 180px;
  height: 180px;
}
.content .el-button {
  width: 60px;
  height: 60px;
  border: none;
  padding: 0;
  background: transparent;
}
.content .el-button-group i {
  font-size: 62px;
  color: #223d54;
}
.content .el-button-group .el-button {
  z-index: 2;
}
.content .el-button-group .el-icon-refresh {
  padding: 10px;
  font-size: 44px;
  color: rgb(212, 226, 245);
  background: #223d54;
  border-radius: 50%;
  z-index: 2;
}
.content:before,
.content:after {
  content: "";
  display: inline-block;
  position: absolute;
  width: 15px;
  height: 212px;
  top: -20px;
  background: #223d54;
  overflow: hidden;
  -webkit-transform: rotate(45deg);
  transform: rotate(45deg);
  left: 50%;
}
.content:after {
  top: -9px;
  -webkit-transform: rotate(-45deg);
  transform: rotate(-45deg);
}
</style>
