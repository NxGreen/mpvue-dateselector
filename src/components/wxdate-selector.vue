<template>
  <picker :class="{active:!placeholderShow}" mode='multiSelector' @columnchange='bindDateColumnChange' @change="bindDateChange" :value='dateIndex' :range='date'>
    {{recentTimeStr}}
  </picker>
</template>

<script>

export default {
  props: {
    beginTime: {
      type: Array,
      default: [1949, 10, 1, 0]
    },
    endTime: {
      type: Array,
      default: function() {
        var now = new Date()
        var year = now.getFullYear()
        var month = now.getMonth() + 1
        var day = now.getDate()
        var hours = now.getHours()
        return [year, month, day, hours]
      }
    },
    selectTime: {
      // 选中时间
      default: [1990, 6, 16, 0]
    },
    defSelectTime: {
      // 默认选中时间
      type: Array,
      default: [1990, 6, 16, 0]
    },
    isUnclear: {
      // 是否显示小时 不清楚选项
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      dateIndex: [1, 1, 1, 1],
      date: [], // 时间数据
      recentTimeStr: '',
      recentTime: [1990, 6, 16, 0], // 存储选中时间 对应 props-selectTime
      placeholderShow: true, // 默认显示 占位符
      placeholder: '公历年月日小时'
    }
  },
  watch: {
    selectTime: function(news, olds) {
      // console.log('selectTime:', news, olds)
      this.init()
    }
  },
  methods: {
    /**
     * 出生日期  change 事件（点击确定后）
     */
    bindDateChange: function(e) {
      // console.log('picker发送选择改变，携带值为', e.mp.detail.value)
      var _recentTime = this.recentTime
      // console.log(_recentTime)
      this.recentTime = _recentTime
      // 传递给组件外使用
      this.placeholderShow = false
      this.recentTimeStr = this.changeTime(_recentTime)
      this.$emit('datechange', _recentTime)
    },
    /**
     * 出生日期 年月日 三级联动
     */
    bindDateColumnChange: function(e) {
      // console.log('修改的列为',e.mp.detail.column,'，值的下标为',e.mp.detail.value)
      var _column = e.mp.detail.column
      var _value = e.mp.detail.value
      var _beginTime = this.beginTime
      var _endTime = this.endTime
      var _recentTime = this.recentTime
      // 变化前 recentTime 下标
      var _curRV = this.dateIndex[_column]
      // 方向 >0向下 <0向上
      var dir = _value - _curRV

      _recentTime[_column] = _recentTime[_column] + dir
      // console.log(_recentTime[_column], _value, _curRV, dir)
      _recentTime[_column] = _recentTime[_column] < 0 ? 0 : _recentTime[_column]
      var bt = new Date(
        _beginTime[0],
        _beginTime[1],
        _beginTime[2],
        _beginTime[3],
        _beginTime[4]
      ).getTime()
      var et = new Date(
        _endTime[0],
        _endTime[1],
        _endTime[2],
        _endTime[3],
        _endTime[4]
      ).getTime()
      var rt = new Date(
        _recentTime[0],
        _recentTime[1],
        _recentTime[2],
        _recentTime[3],
        _recentTime[4]
      ).getTime()
      if (rt < bt) {
        _recentTime = _beginTime.concat()
      }
      if (rt > et) {
        _recentTime = _endTime.concat()
      }
      var _data = this.makeDate(_recentTime)

      this.date = _data.date
      this.dateIndex = _data.dateIndex
      this.recentTime = _data.recentTime
      this.placeholderShow = false
      this.recentTimeStr = this.changeTime(_recentTime)
      this.$emit('datechange', _recentTime)
    },
    changeTime: function(recentTime) {
      // console.log(this.placeholderShow)
      if (this.placeholderShow) {
        return this.placeholder
      }
      var arr = []
      for (var i = 0; i < recentTime.length; i++) {
        var temp = recentTime[i]
        if (i == 3 && this.isUnclear) {
          // 处理时间不清楚显示
          temp = temp == 0 ? '不清楚' : temp - 1
        }
        temp += ''
        if (temp.length == 1) {
          temp = '0' + temp
        }
        arr.push(temp)
      }
      // console.log(arr)
      if (this.isUnclear && arr[3] == '不清楚') {
        return arr[0] + '-' + arr[1] + '-' + arr[2] + ' ' + arr[3]
      } else {
        return arr[0] + '-' + arr[1] + '-' + arr[2] + ' ' + arr[3] + ':00'
      }
    },
    /**
     * eg:this.buildArr(1990,2018,'年')
     */
    buildArr: function(_star, _end, company, _isUnclear) {
      var arr = []
      var end = Math.max(_star, _end)
      var star = Math.min(_star, _end)
      for (var i = star; i <= end; i++) {
        arr.push(i + company)
      }
      if (_isUnclear) {
        arr.unshift('不清楚')
      }
      return arr
    },
    loop: function(begin, length, fuc) {
      for (var i = begin; i < length; i++) {
        if (fuc(i)) break
      }
    },
    checkTimeArr: function(arr1, arr2, length) {
      var checkStatus = true
      this.loop(0, length, function(i) {
        if (arr1[i] !== arr2[i]) checkStatus = false
      })
      return checkStatus
    },
    /**
     * 初始化 date 日期时间二维数组
     */
    makeDate: function(recentTime) {
      var arr = []
      var that = this
      var beginTime = this.beginTime
      var endTime = this.endTime
      var dateIndex = []
      this.loop(0, beginTime.length, function(i) {
        var min = 0
        var max = 0
        switch (i) {
          case 0:
            arr.push(that.buildArr(beginTime[i], endTime[i], '年'))
            dateIndex[i] = recentTime[i] - beginTime[i]
            break
          case 1:
            min = that.checkTimeArr(beginTime, recentTime, 1) ? beginTime[i] : 1
            max = that.checkTimeArr(endTime, recentTime, 1) ? endTime[i] : 12
            // console.log(min+':'+max);
            dateIndex[i] = that.checkTimeArr(beginTime, recentTime, 1)
              ? recentTime[i] - beginTime[i]
              : recentTime[i] - 1
            arr.push(that.buildArr(min, max, '月'))
            break
          case 2:
            min = that.checkTimeArr(beginTime, recentTime, 2) ? beginTime[i] : 1
            max = that.checkTimeArr(endTime, recentTime, 2)
              ? endTime[i]
              : new Date(recentTime[0], recentTime[1], 0).getDate()
            // console.log(min + ':' + max);
            dateIndex[i] = that.checkTimeArr(beginTime, recentTime, 2)
              ? recentTime[i] - beginTime[i]
              : recentTime[i] - 1
            arr.push(that.buildArr(min, max, '日'))
            break
          case 3:
            min = that.checkTimeArr(beginTime, recentTime, 3) ? beginTime[i] : 0
            max = that.checkTimeArr(endTime, recentTime, 3) ? endTime[i] : 23
            // console.log(min + ':' + max);
            dateIndex[i] = that.checkTimeArr(beginTime, recentTime, 3)
              ? recentTime[i] - beginTime[i]
              : recentTime[i]
            arr.push(that.buildArr(min, max, '时', that.isUnclear))
            break
          case 4:
            min = that.checkTimeArr(beginTime, recentTime, 4) ? beginTime[i] : 0
            max = that.checkTimeArr(endTime, recentTime, 4) ? endTime[i] : 59
            // console.log(min + ':' + max);
            dateIndex[i] = that.checkTimeArr(beginTime, recentTime, 4)
              ? recentTime[i] - beginTime[i]
              : recentTime[i]
            arr.push(that.buildArr(min, max, '分'))
            break
          case 5:
            min = that.checkTimeArr(beginTime, recentTime, 5) ? beginTime[i] : 0
            max = that.checkTimeArr(endTime, recentTime, 5) ? endTime[i] : 59
            // console.log(min + ':' + max);
            dateIndex[i] = that.checkTimeArr(beginTime, recentTime, 5)
              ? recentTime[i] - beginTime[i]
              : recentTime[i]
            arr.push(that.buildArr(min, max, '秒'))
            break
        }
      })
      // console.log(dateIndex)
      // console.log(recentTime)
      return {
        date: arr,
        dateIndex: dateIndex,
        recentTime: recentTime
      }
    },
    init () {
      if (!this.selectTime) {
        this.recentTime = this.defSelectTime.concat()
      } else {
        this.placeholderShow = false
        this.recentTime = this.selectTime.concat()
      }

      var _data = this.makeDate(this.recentTime)
      this.date = _data.date
      this.dateIndex = _data.dateIndex
      this.recentTime = _data.recentTime
      this.recentTimeStr = this.changeTime(_data.recentTime)
    }
  },
  created () {
    this.init()
  }
}
</script>


<style scoped>
.picker {
  display: flex;
  align-items: center;
  height: 130rpx;
  line-height: normal;
  flex-wrap: nowrap;
}

.active {
  color: #333;
}
</style>
