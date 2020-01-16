
<template >
  <div v-if="unRegister">
    <x-header :left-options="{showBack: false}" class="header">稽查登录</x-header>
    <group title="请使用用户名和密码登录">
      <x-input title="用户名" placeholder="请输入用户名" v-model="card" required></x-input>
      <x-input title="密码" type="password" placeholder="请输入密码" v-model="password" required></x-input>
    </group>
    <div style="margin-top:50px">
      <x-button type="primary" @click.native="addClient">登录</x-button>
    </div>
  </div>
  <div v-else>
    <x-header :left-options="{showBack: false}" class="header">稽查信息</x-header>
    <group gutter="0">
      <search @on-change="filterList" v-model="searchPlate" top="45px" placeholder="可以通过车牌号搜索"></search>
    </group>
    <group gutter="0">
      <cell title="门卫稽查" v-show="inspect=='总稽查'||inspect=='门卫稽查'">
        <div>
          <checker
            v-model="door"
            default-item-class="demo1-item"
            selected-item-class="demo1-item-selected"
            @on-change="filterList"
          >
            <checker-item value="1">是</checker-item>
            <checker-item value="2">否</checker-item>
            <checker-item value="3">全部</checker-item>
          </checker>
        </div>
      </cell>
      <cell title="保安稽查" v-show="inspect=='总稽查'||inspect=='保安稽查'">
        <div>
          <checker
            v-model="ensure"
            @on-change="filterList"
            default-item-class="demo1-item"
            selected-item-class="demo1-item-selected"
          >
            <checker-item value="1">是</checker-item>
            <checker-item value="2">否</checker-item>
            <checker-item value="3">全部</checker-item>
          </checker>
        </div>
      </cell>
      <cell title="交易类型">
        <div>
          <checker
            v-model="chargeType"
            default-item-class="demo1-item"
            selected-item-class="demo1-item-selected"
            @on-change="filterList"
          >
            <checker-item value="1">卸货</checker-item>
            <checker-item value="2">交易</checker-item>
            <checker-item value="3">全部</checker-item>
          </checker>
        </div>
      </cell>
      <cell title="品种大类">
        <div>
          <checker
            v-model="goodsAllType"
            default-item-class="demo1-item"
            selected-item-class="demo1-item-selected"
            @on-change="filterList"
          >
            <checker-item value="1">水果</checker-item>
            <checker-item value="2">蔬菜</checker-item>
            <checker-item value="3">全部</checker-item>
          </checker>
        </div>
      </cell>
      <cell title="交易区域">
        <div>
          <checker
            v-model="chargeArea"
            default-item-class="demo1-item"
            selected-item-class="demo1-item-selected"
            @on-change="filterList"
          >
            <checker-item v-for="item in chargeAreaList" :value="item">{{item}}</checker-item>

            <checker-item value="全部">全部</checker-item>
          </checker>
        </div>
      </cell>
    </group>

    <div
      class="box"
      slot="content"
      @click="showAppoint(item)"
      v-for="item in showList"
      :key="item.id"
    >
      <flexbox>
        <flexbox-item>
          <div class="owner">{{item.plateNumber}}</div>
        </flexbox-item>
        <flexbox-item>
          <div class="changeType">{{item.changeType}}</div>
        </flexbox-item>
      </flexbox>
      <flexbox>
        <flexbox-item>
          <div class="address">{{item.client}}</div>
        </flexbox-item>
        <flexbox-item>
          <div class="changeType">{{item.enterTime}}</div>
        </flexbox-item>
      </flexbox>
    </div>

    <!--
    <div class="round-click" @click="addCar">
      <a>+</a>
    </div>
    -->
  </div>
</template>

<script>
import {
  Tabbar,
  TabbarItem,
  Group,
  Cell,
  XHeader,
  CellBox,
  Panel,
  AjaxPlugin,
  TransferDom,
  Previewer,
  XButton,
  XInput,
  XTextarea,
  XDialog,
  FormPreview,
  Flexbox,
  FlexboxItem,
  Msg,
  Tab,
  TabItem,
  GroupTitle,
  Datetime,
  Selector,
  Grid,
  GridItem,
  Swipeout,
  SwipeoutItem,
  SwipeoutButton,
  Icon,
  Checker,
  CheckerItem,
  Search
} from "vux";
import GoodsList from "./GoodsList";
import HttpUtil from "../util/HttpUtil";
import MyUtil from "../util/MyUtil";

export default {
  directives: {
    TransferDom
  },
  data() {
    let data = [];

    return {
      unRegister: true,
      birthday: "1990-01-01",
      sexList: ["男", "女"],
      sex: "男",
      card: "",
      password: "",
      client: "",
      tel: "",
      headImgUrl: "",
      couponsCount: 0,
      money: 0,
      integral: 0,
      goodsType: [],
      showList: [],
      appointList: [],
      inspect: "",
      door: "3",
      ensure: "3",
      boss: "3",
      chargeType: "3",
      searchPlate: "",
      goodsAllType: "3",
      chargeArea: "全部",
      chargeAreaList: []
    };
  },
  mounted() {
    let _this = this;
    console.log(this.$store.state.vux.snsUserInfo);
    if (!this.$store.state.vux.snsUserInfo.username) {
      if (this.$store.state.vux.snsUserInfo.openId) {
        _this.unRegister = true;
        _this.wxid = _this.$store.state.vux.snsUserInfo.openId;
        return;
      }
      let code = _this.$geturlpara.getUrlKey("code");
      let state = _this.$geturlpara.getUrlKey("state");
      let username = _this.$geturlpara.getUrlKey("username");
      this.$store.commit("updateTenant", { tenant: username });
      HttpUtil.get(
        "wechat/clientInspectLogin.do",
        { code: code, state: state },
        function(data) {
          _this.wxid = data.openId;
          _this.headImgUrl = data.headImgUrl;
          _this.$store.commit("updateSnsUserInfo", data);

          _this.sex = data.sex == 1 ? "男" : "女";
          if (data.username != "null") {
            _this.unRegister = false;
            _this.$store.commit("updateUsername", data.username);
            _this.inspect = data.clientPrincipal;
            _this.client = data.username;
            _this.getApponitList();
          }
        }
      );

      /*
      HttpUtil.get('wechat/getWechatUser',{},function(data){

      })
      */
    } else {
      if (_this.$store.state.vux.snsUserInfo.username != "null") {
        _this.unRegister = false;
        _this.wxid = _this.$store.state.vux.snsUserInfo.openId;

        _this.client = _this.$store.state.vux.snsUserInfo.username;
        _this.inspect = _this.$store.state.vux.snsUserInfo.clientPrincipal;
        _this.number = _this.$store.state.vux.snsUserInfo.tel;
        _this.headImgUrl = _this.$store.state.vux.snsUserInfo.headImgUrl;
        _this.money = _this.$store.state.vux.snsUserInfo.advanceGathering;
        _this.integral = _this.$store.state.vux.snsUserInfo.integral;
        _this.getApponitList();
      }
    }
    this.getAreaList();
    this.door = localStorage.getItem("door")
      ? localStorage.getItem("door")
      : "3";
    this.ensure = localStorage.getItem("ensure")
      ? localStorage.getItem("ensure")
      : "3";
    this.chargeType = localStorage.getItem("chargeType")
      ? localStorage.getItem("chargeType")
      : "3";
    this.chargeArea = localStorage.getItem("chargeArea")
      ? localStorage.getItem("chargeArea")
      : "全部";
    this.goodsAllType = localStorage.getItem("goodsAllType")
      ? localStorage.getItem("goodsAllType")
      : "3";
  },
  methods: {
    showLoading() {
      this.$vux.loading.show({
        text: "正在登录"
      });
    },
    hideLoading() {
      this.$vux.loading.hide();
    },
    getApponitList() {
      let _this = this;
      HttpUtil.get("wechat/getCarList.do", { client: this.client }, function(
        data
      ) {
        _this.appointList = data;
        _this.showList = data;
        _this.filterList();
      });
    },
    filterList() {
      let _this = this;
      localStorage.setItem("door", this.door);
      localStorage.setItem("ensure", this.ensure);
      localStorage.setItem("chargeType", this.chargeType);
      localStorage.setItem("chargeArea", this.chargeArea);
      localStorage.setItem("goodsAllType", this.goodsAllType);
      this.showList = this.appointList.filter(function(item) {
        let result = true;
        if (_this.searchPlate != "") {
          result = result && item.plateNumber.indexOf(_this.searchPlate) != -1;
        }
        if (_this.inspect == "总稽查" || _this.inspect == "门卫稽查") {
          if (_this.door == "1") {
            result = result && item.door != "";
          } else if (_this.door == "2") {
            result = result && item.door == "";
          }
        }
        if (_this.inspect == "总稽查" || _this.inspect == "保安稽查") {
          if (_this.ensure == "1") {
            result = result && item.ensure != "";
          } else if (_this.ensure == "2") {
            result = result && item.ensure == "";
          }
        }
        if (_this.chargeType == "1") {
          result = result && item.chargeType == "卸货";
        } else if (_this.chargeType == "2") {
          result = result && item.chargeType == "交易";
        }

        if (_this.chargeArea != "全部") {
          result = result && item.chargeArea == _this.chargeArea;
        }

        if (_this.goodsAllType == "1") {
          result = result && item.goodsAllType == "水果";
        } else if (_this.goodsAllType == "2") {
          result = result && item.goodsAllType == "蔬菜";
        }

        return result;
      });
    },
    addCar() {
      this.$router.push({ path: "/addCar" });
    },
    delAppoint(item) {
      let _this = this;

      HttpUtil.get(
        "wechat/delAppoint.do",
        { billCode: item.billCode },
        function(data) {
          _this.getApponitList();
        }
      );
    },
    showAppoint(item) {
      console.log(item.billCode);
      this.$router.push({
        name: "addCar",
        params: { billCode: item.serial }
      });
    },
    getAreaList() {
      let _this = this;
      HttpUtil.get("wechat/getAreaList.do", {}, function(data) {
        _this.chargeAreaList = [];
        data.forEach(element => {
          _this.chargeAreaList.push(element["交易区域"].trim());
        });
        console.log(_this.chargeAreaList);
      });
    },
    addClient() {
      if (!this.card) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入用户名"
        });
        return;
      }

      if (!this.wxid) {
        this.$vux.alert.show({
          title: "提示",
          content: "页面状态不对，请关闭重新进入"
        });
        return;
      }
      let _this = this;
      console.log(this.birthday);
      this.showLoading();
      HttpUtil.post(
        "wechat/inspectLogin.do",
        {
          username: this.card,
          password: this.password,
          openid: this.wxid
        },
        function(data) {
          console.log(data);
          _this.hideLoading();
          if (data.result == "success") {
            _this.$store.commit("updateUsername", data.result1);
            _this.$store.commit("updateTenant", data.result2);
            _this.client = data.result1;
            _this.inspect = data.result2;
            _this.unRegister = false;
          } else if (data.result == "noAuth") {
            _this.$vux.toast.show({
              text: "该操作员没有稽查权限",
              type: "warn"
            });
          } else {
            _this.$vux.toast.show({
              text: "用户名或密码错误",
              type: "warn"
            });
          }
        },
        function(error) {
          _this.hideLoading();
        }
      );
    }
  },
  components: {
    Tabbar,
    TabbarItem,
    Group,
    Cell,
    XHeader,
    CellBox,
    Panel,
    GoodsList,
    Previewer,
    XButton,
    XInput,
    XTextarea,
    XDialog,
    FormPreview,
    Flexbox,
    FlexboxItem,
    Msg,
    Tab,
    TabItem,
    GroupTitle,
    Datetime,
    Selector,
    Grid,
    GridItem,
    Selector,
    Swipeout,
    SwipeoutItem,
    SwipeoutButton,
    Icon,
    Checker,
    CheckerItem,
    Search
  }
};
</script>
<style lang="scss" scoped>
.demo1-item {
  border: 1px solid #ececec;
  padding: 5px 15px;
  font-size: 12px;
}
.demo1-item-selected {
  border: 1px solid green;
}
.dr-user-info {
  width: 100%;
  height: 6.7667rem;
  position: relative;
  text-align: center;
  color: #000;
  background: url(../assets/img/user_bg.jpg) no-repeat top center;
  background-size: 100%;
}
.dr-user-info .avatar {
  width: 2.9733rem;
  height: 2.9733rem;
  border-radius: 50%;
  padding: 0.3867rem;
  overflow: hidden;
  margin: 0.5467rem 0.0933rem;
  background: #e7e7e7;
}
.dr-user-info .avatar .avatar-wrap {
  background: #a9cfa9;
  width: 2.4rem;
  height: 2.4rem;
  padding: 0.18rem;
  border-radius: 50%;
}
.dr-user-info .avatar .avatar-wrap img {
  width: 2.3733rem;
  height: 2.3733rem;
  border-radius: 50%;
  border: 1px solid #1a961a;
}
.dr-user-info .username {
  color: #689524;
  font-size: 16px;
}
.dr-user-info .userid {
  font-size: 0.6467rem;
  line-height: 1;
  margin-top: 0.5rem;
  position: relative;
  padding-right: 1.3133rem;
  display: inline-block;
  bottom: -1rem;
}
.logo {
  width: 150px;

  margin: 10px auto 0;
}
.vip {
  position: absolute;
  right: 10px;
  bottom: 20px;
  color: red;
}

.box {
  border-bottom: 1px solid #ececec;
  padding: 10px 10px 9px;

  background-color: #fff;
  position: relative;
}
.car {
  font-size: 14px;
  line-height: 15px;
  color: #222;
  height: 15px;
  overflow: hidden;
}
.owner {
  font-size: 15px;
  line-height: 17px;
  height: 17px;
  color: #999;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  margin: 3px 0 4px;
}
.owner span {
  font-size: 8px;
  padding: 0 5px;
  position: relative;
  top: -2px;
  color: #e1e1e1;
}
.changeType {
  font-size: 15px;
  color: #999;
  text-align: right;
  padding-right: 20px;
}
.address {
  font-size: 15px;
  line-height: 17px;
  height: 17px;
  color: #999;

  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  margin: 3px 0 4px;
}
.deal {
  position: absolute;
  right: 0;
  top: 15px;
}

.round-click {
  height: 40px;
  width: 40px;
  background-color: #1aad19;
  opacity: 0.7;
  padding: 0;
  position: fixed;
  bottom: 75px;
  right: 20px;
  border-radius: 100%;
  line-height: 40px;
  text-align: center;
  z-index: 90;
}
.round-click a {
  font-size: 42px;
  max-width: 40px;
  color: #fff;
}
.header {
  background-color: #1aad19;
}
</style>
