# mpvue-dateselector

> mpvue基于微信小程序 开发的 时间选择器(可以精确到 小时分秒)
如果需要微信小程序原生 时间选择器 请 [wxDateSelector](https://github.com/NxGreen/wxDateSelector) 


## Build Setup

``` bash
npm install mpvue-dateselector --save
```

## Usage

``` bash
# 安装
<template>
  <div>
    <wxdate-selector :selectTime="selectTime" @datechange="datechange"></wxdate-selector>
  </div>
</template>

<script>
import wxdateSelector from '@/components/wxdate-selector'

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
    }
  }
}
</script>
```

### 参数配置

| 参数 | 字符类型  |  取值  | 说明 | 
| -----| -----| -----| -----|
|  **beginTime**   |  {Array} |*eg:[2000, 1, 1, 0, 0, 0]    2000年1月1日0时0分0秒*| 设置开始时间点 |
|  **endTime** |  {Array} |同beginTime| 设置结束时间点 |
|  **selectTime**  |  {Array} |同beginTime| 设置选中时间点  |
|  **defSelectTime**  |  {Array} |同beginTime| 设置默认选中时间点（当有占位符显示时，可以设置一个默认选中时间） |
|  **isUnclear**  |  {Boolean} |false| 小时增加一个不清楚选项字段（公司内部需求） |


### 注意事项
 1. 受限于mpvue的组件限制，当子组件（wxdate-selector）chang事件触发通过$emit把selectTime值返回给父组件时，如果直接赋值给data上的selectTime时(即实现组件双向绑定数据时)，会导致微信小程序下拉效果会跳动。暂且把selectTime保存到临时变量中解决跳动问题。具体原因不知道为何，还望有知青人士告知原因。
