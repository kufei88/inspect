<template>
  <div>
    <x-header :left-options="{showBack: true}" class="header">现场稽查</x-header>
    <group>
      <x-input
        title="车牌"
        v-model="plateNumber"
        readonly="readonly"
        required
        placeholder="请输入车牌,挂车请输挂车号"
      ></x-input>
      <selector title="交易类型" v-model="changeType" :options="changeTypeList"></selector>
      <selector
        v-show="changeType=='交易'"
        title="品种大类"
        v-model="goodsAllType"
        :options="goodsAllTypeList"
        @on-change="typeChange"
      ></selector>
      <selector
        v-show="goodsAllType=='水果'&&changeType=='交易'"
        title="交易区域"
        v-model="changeArea"
        :options="changeAreaList"
      ></selector>
      <x-number title="车辆吨位" min="0" v-model="dun" fillable></x-number>
      <!--  <selector title="品种" v-model="goodsType" :options="goodTypeList"></selector>  -->
      <cell title="品种" :value="goodsType" is-link="true" @click.native="showSelect"></cell>
      <x-input
        title="驾驶员联系电话"
        v-model="number"
        keyboard="number"
        is-type="china-mobile"
        placeholder="请输入驾驶员电话"
      ></x-input>

      <cell title="产地" :value="address"></cell>
      <x-number title="产品吨位" min="0" v-model="goodsDun" fillable></x-number>
      <datetime v-model="appointTime" format="YYYY-MM-DD HH:mm" title="进场时间"></datetime>
      <x-textarea title="备注" v-model="remark"></x-textarea>
    </group>
    <x-dialog v-model="showSelectType">
      <group>
        <x-input placeholder="用拼音码定位" :value="search" @on-change="searchChange"></x-input>
      </group>
      <scroller lock-x scrollbar-y height="400px" ref="scroller">
        <group>
          <cell
            :title="item.value"
            v-for="item in showGoodsTypeList"
            @click.native="selectGoodsType(item.value)"
          ></cell>
        </group>
      </scroller>
    </x-dialog>
    <group>
      <x-button type="primary" @click.native="save">保存</x-button>
    </group>
  </div>
</template>

<script>
import ChinaAddressV4Data from "./china_address.json";
import {
  Group,
  Cell,
  XHeader,
  CheckIcon,
  Search,
  XInput,
  Checker,
  CheckerItem,
  Divider,
  XSwitch,
  XTextarea,
  XButton,
  Selector,
  XAddress,
  Datetime,
  XNumber,
  Value2nameFilter as value2name,
  XDialog,
  Scroller
} from "vux";
import HttpUtil from "../util/HttpUtil";
let pinyin = require("js-pinyin");
export default {
  components: {
    Group,
    Cell,
    XHeader,
    CheckIcon,
    Search,
    XInput,
    Checker,
    CheckerItem,
    Divider,
    XSwitch,
    XTextarea,
    XButton,
    Selector,
    XAddress,
    Datetime,
    XNumber,
    XDialog,
    Scroller
  },
  name: "addcar",
  data() {
    return {
      car: "",
      caruser: "",
      number: "",
      address: [],
      remark: "",
      margin: false,
      goodTypeList: [],
      showGoodsTypeList: [],
      changeType: "",
      oldChangeType: "",
      goodsAllType: "",
      changeArea: "",
      changeTypeList: ["卸货", "交易"],
      goodsAllTypeList: ["水果", "蔬菜"],
      changeAreaList: ["中央操场", "东关"],
      plateNumber: "",
      dun: 0.0,
      goodsType: "",
      addressData: ChinaAddressV4Data,
      appointTime: "",
      goodsDun: 0.0,
      state: 0,
      showSelectType: false,
      search: ""
    };
  },
  mounted() {
    console.log(JSON.stringify(this.$route.params));
    if (this.$route.params.billCode) {
      this.state = 1;
      this.getAppointByBillCode();
    }
    this.getGoodTypeList("");
    this.getAreaList();
  },
  methods: {
    typeChange(value) {
      console.log(value);
      this.getGoodTypeList(value);
    },
    showSelect() {
      this.showGoodsTypeList = this.goodTypeList;
      this.showSelectType = true;
      this.search = "";
    },
    searchChange(value) {
      this.showGoodsTypeList = this.goodTypeList.filter(function(item) {
        let pycode = pinyin.getCamelChars(item.value);

        return (
          pycode.indexOf(value.toUpperCase()) != -1 ||
          item.value.indexOf(value) != -1
        );
      });
    },
    selectGoodsType(value) {
      this.goodsType = value;
      this.showSelectType = false;
    },
    getAreaList() {
      let _this = this;
      HttpUtil.get("wechat/getAreaList.do", {}, function(data) {
        _this.changeAreaList = [];
        data.forEach(element => {
          _this.changeAreaList.push(element["交易区域"].trim());
        });
      });
    },
    getAppointByBillCode() {
      let _this = this;
      HttpUtil.get(
        "wechat/getCarByBillCode.do",
        { billCode: this.$route.params.billCode },
        function(data) {
          _this.plateNumber = data.plateNumber;
          _this.dun = parseFloat(data.dun);
          _this.goodsType = data.goodsType;
          _this.number = data.number;
          _this.address = data.address;
          _this.goodsDun = parseFloat(data.goodsDun);
          _this.remark = data.remark;
          _this.changeType = data.changeType;
          _this.oldChangeType = data.changeType;
          _this.changeArea = data.changeArea;

          _this.goodsAllType = data.goodsAllType;
          _this.appointTime = data.appointTime;
        }
      );
    },
    getGoodTypeList(allType) {
      let _this = this;
      HttpUtil.get("wechat/getGoodsTypeList.do", { allType: allType }, function(
        data
      ) {
        _this.goodTypeList = data;
        _this.showGoodsTypeList = data;
      });
    },
    save() {
      console.log(this.goodsType);
      if (!this.plateNumber) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入车牌号"
        });
        return;
      }

      if (!this.dun) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入车辆吨位"
        });
        return;
      }
      if (!this.goodsType) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入品种"
        });
        return;
      }
      if (!this.number) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入驾驶员联系电话"
        });
        return;
      }
      if (this.address.length == 0) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入产地"
        });
        return;
      }
      if (!this.goodsDun) {
        this.$vux.alert.show({
          title: "提示",
          content: "请输入产品吨位"
        });
        return;
      }
      if (!this.appointTime) {
        this.$vux.alert.show({
          title: "提示",
          content: "请选择预约进场时间"
        });
        return;
      }
      if (!this.$store.state.vux.snsUserInfo.username) {
        this.$vux.alert.show({
          title: "提示",
          content: "页面状态不对，请关闭重新进入"
        });
        return;
      }
      if (!this.changeType) {
        this.$vux.alert.show({
          title: "提示",
          content: "请选择交易类型"
        });
        return;
      }
      if (this.changeType == "交易" && !this.goodsAllType) {
        this.$vux.alert.show({
          title: "提示",
          content: "请选择品种大类"
        });
        return;
      }
      if (
        this.changeType == "交易" &&
        this.goodsAllType == "水果" &&
        !this.changeArea
      ) {
        this.$vux.alert.show({
          title: "提示",
          content: "请选择交易区域"
        });
        return;
      }
      let _this = this;
      let postAppoint = {
        plateNumber: this.plateNumber,
        dun: this.dun,
        goodsType: this.goodsType,
        number: this.number,
        address: this.address,
        goodsDun: this.goodsDun,
        remark: this.remark,
        changeType: this.changeType,
        changeArea:
          this.changeType == "交易" && this.goodsAllType == "水果"
            ? this.changeArea
            : "",
        goodsAllType: this.changeType == "交易" ? this.goodsAllType : "",
        appointTime: this.appointTime,
        client: this.$store.state.vux.snsUserInfo.username,
        billCode: this.$route.params.billCode ? this.$route.params.billCode : ""
      };
      HttpUtil.post(
        "wechat/addInspect.do",
        {
          appoint: JSON.stringify(postAppoint),
          oldChangeType: this.oldChangeType,
          state: this.$store.state.vux.snsUserInfo.clientPrincipal
        },
        function(data) {
          if (data == 0) {
            _this.$vux.alert.show({
              title: "提示",
              content: "该车已存在"
            });
            return;
          } else if (data == -1) {
            _this.$vux.toast.show({
              text: "不能由交易改为卸货"
            });
          } else if (data == -2) {
            _this.$vux.toast.show({
              text: "停车费不能由高到低"
            });
          } else if (data > 0) {
            _this.$vux.toast.show({
              text: "稽查成功"
            });
            _this.$router.back();
          }
        }
      );
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}

.demo1-item {
  border: 1px solid #ececec;
  padding: 5px 15px;
}
.demo1-item-selected {
  border: 1px solid green;
}
.box {
  padding: 0 15px;
}
.header {
  background-color: #1aad19;
}
</style>
