<template>
  <div>
    <div class="line">
      <div class="txt">出生时间</div>
      <div class="wrap-input">
        <wxdate-selector :selectTime="selectTime" @datechange="datechange"></wxdate-selector>
      </div>
    </div>
    <div class="line">
      <button class='btn' @click="gopay">提交</button>
    </div>
  </div>
</template>

<script>
import wxdateSelector from '@/components/wxdate-selector'
/* 貌似组件双向绑定导致 下拉跳动，（甚至父组件数据更新会影响 子组件的数据更新，临时保存在data中也不行）
  so，用_formdata于接受子组件返回时间(地点)的数据更新 保存到内存中，然后 存储到本地使用 （注意切页面保存到本地，和获取本地数据更新）
  1.在data中设置selectTime会导致下拉跳动
  2.当然也可以用vuex 来保存数据到全局 但需要做持久化到本地保存
  暂且用此方法解决 下拉跳动问题 */
let selectTime = '' // 同selectTime 数据格式完全相同 下面同样如此
export default {
  data() {
    return {
      selectTime: ''
    }
  },
  components: {
    wxdateSelector
  },
  methods: {
    datechange(e) {
      console.log(e)
      selectTime = e
    },
    gopay() {
      // 验证日期
      if (!selectTime || selectTime == '公历年月日小时') {
        wx.showToast({ title: '请填写时间' })
        return false
      } else {
        wx.setStorageSync('selectTime', selectTime)
        wx.showToast({ title: '保存到本地成功' })
      }
    },
    getInitData() {
      var _selectTime = wx.getStorageSync('selectTime')
      if (_selectTime) {
        this.selectTime = _selectTime
      }
    }
  },
  created() {
    this.getInitData()
  },
  onShow() {
    this.getInitData()
  }
}
</script>
<style>
.line {
  position: relative;
  margin: 0 1rem;
  height: 0.8rem;
  line-height: 0.8rem;
  border-bottom: 1px solid #ffe5e9;
}
.txt {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
  width: 1.7rem;
  text-align: left;
  font-size: 0.28rem;
  color: #333;
}
.wrap-input {
  height: 0.8rem;
  line-height: 0.82rem;
  padding: 0 0 0 1.7rem;
  font-size: 0.28rem;
  color: #ccc;
}
.btn {
  width: 5.92rem;
  height: 0.9rem;
  line-height: 0.9rem;
  margin: 0.36rem auto;
  font-size: 0.36rem;
  font-weight: bold;
  text-align: center;
  color: #fff;
  border-radius: 0.2rem;
  background: #fe84b5;
  background: -webkit-linear-gradient(-45deg, #fe84b5, #ff7791);
  background: linear-gradient(-45deg, #fe84b5, #ff7791);
}
</style>
