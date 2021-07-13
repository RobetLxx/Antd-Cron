<template>
  <a-card class="easy-cron">
    <div class="content">
      <div class="left">
        <a-tabs size="small" v-model="curtab">
          <a-tab-pane tab="秒" key="second" v-if="!hideSecond"><second-ui v-model="second" :disabled="disabled"></second-ui></a-tab-pane>
          <a-tab-pane tab="分" key="minute"><minute-ui v-model="minute" :disabled="disabled"></minute-ui></a-tab-pane>
          <a-tab-pane tab="时" key="hour"><hour-ui v-model="hour" :disabled="disabled"></hour-ui></a-tab-pane>
          <a-tab-pane tab="日" key="day"><day-ui v-model="day" :week="week" :disabled="disabled"></day-ui></a-tab-pane>
          <a-tab-pane tab="周" key="week"><week-ui v-model="week" :day="day" :disabled="disabled"></week-ui></a-tab-pane>
          <a-tab-pane tab="月" key="month"><month-ui v-model="month" :disabled="disabled"></month-ui></a-tab-pane>
          <a-tab-pane tab="年" key="year" v-if="!hideYear && !hideSecond"><year-ui v-model="year" :disabled="disabled"></year-ui></a-tab-pane>
        </a-tabs>
      </div>
      <div class="right">
        <div class="field-list">
          <a-table
            ref="table"
            rowkey="name"
            :loading="loading"
            :columns="columns"
            :dataSource="tableData"
            :pagination="false"
            size="small"
          >
          </a-table>
        </div>
        <div class="exe-pre">
          <div class="exe-pre-panel">
            <label class="p-left">执行时间</label>
            <a-date-picker showTime v-model="startTime" class="p-right"
                           placeholder="选择执行开始时间"></a-date-picker>
          </div>
          <div class="exe-pre-panel">
            <a-tooltip content="执行预览解析不含年参数" class="p-left">
              <label>执行预览</label>
            </a-tooltip>
            <a-input type="textarea" :value="preTimeList" class="p-right" :rows="4" readonly />
          </div>
        </div>
      </div>
    </div>
  </a-card>
</template>

<script>
import SecondUi from './tabs/second'
import MinuteUi from './tabs/minute'
import HourUi from './tabs/hour'
import DayUi from './tabs/day'
import WeekUi from './tabs/week'
import MonthUi from './tabs/month'
import YearUi from './tabs/year'
import CronParser from 'cron-parser'
import dateFormat from './format-date'
import { debounce } from 'debounce'
import moment from 'moment'

export default {
  name: 'EasyCron',
  model: {
    prop: 'cronValue',
    event: 'change'
  },
  props: {
    cronValue: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    exeStartTime: {
      type: [Number, String, Object],
      default: 0
    },
    hideSecond: {
      type: Boolean,
      default: false
    },
    hideYear: {
      type: Boolean,
      default: false
    },
    remote: {
      type: Function,
      default: null
    }
  },
  data () {
    return {
      curtab: this.hideSecond ? 'minute' : 'second',
      second: '*',
      minute: '*',
      hour: '*',
      day: '*',
      month: '*',
      week: '?',
      year: '*',
      startTime: moment(new Date(),'YYYY-MM-dd'),
      preTimeList: '执行预览，会忽略年份参数',
      columns: [{ title: '', width: '80', dataIndex: 'name' }, { title: '', dataIndex: 'value' }],
      loading: false,
    }
  },
  computed: {
    tableData () {
      let c = this.hideSecond ? [] : [{ name: '秒', value: this.second }]
      c = c.concat([
        { name: '分', value: this.minute },
        { name: '时', value: this.hour },
        { name: '日', value: this.day },
        { name: '月', value: this.month },
        { name: '周', value: this.week }
      ])
      return (this.hideSecond || this.hideYear) ? c.concat({ name: '表达式', value: this.cronValue_c })
        : c.concat(
          { name: '年', value: this.year },
          { name: '表达式', value: this.cronValue_c },
          { name: '表达式(不含年)', value: this.cronValue_c2 }
        )
    },
    cronValue_c () {
      var result = []
      if (!this.hideSecond) result.push(this.second ? this.second : '*')
      result.push(this.minute ? this.minute : '*')
      result.push(this.hour ? this.hour : '*')
      result.push(this.day ? this.day : '*')
      result.push(this.month ? this.month : '*')
      result.push(this.week ? this.week : '?')
      if (!this.hideYear && !this.hideSecond) result.push(this.year ? this.year : '*')
      return result.join(' ')
    },
    cronValue_c2 () {
      const v = this.cronValue_c
      if (this.hideYear || this.hideSecond) return v
      const vs = v.split(' ')
      return vs.slice(0, vs.length - 1).join(' ')
    }
  },
  watch: {
    cronValue (newVal, oldVal) {
      // console.log(oldVal)
      if (newVal === this.cronValue_c) {
        console.info('same cron value: ' + newVal)
        return
      }
      this.formatValue()
    },
    cronValue_c (newVal, oldVal) {
      this.calTriggerList()
      this.$emit('change', newVal)
    },
    exeStartTime (newVal, oldVal) {
      this.calStartTime()
    },
    startTime (newVal, oldVal) {
      this.calTriggerList()
    }
  },
  methods: {
    formatValue () {
      // console.info(this.cronValue)
      if (!this.cronValue) return
      const values = this.cronValue.split(' ').filter(item => !!item)
      if (!values || values.length <= 0) return
      let i = 0
      if (!this.hideSecond) this.second = values[i++]
      if (values.length > i) this.minute = values[i++]
      if (values.length > i) this.hour = values[i++]
      if (values.length > i) this.day = values[i++]
      if (values.length > i) this.month = values[i++]
      if (values.length > i) this.week = values[i++]
      if (values.length > i) this.year = values[i]
    },
    calTriggerList: debounce(function () {
      this.calTriggerListInner()
    }, 10),
    calTriggerListInner () {
      // 设置了回调函数
      if (this.remote) {
        this.remote(this.cronValue_c2, +this.startTime, v => { this.preTimeList = v })
        return
      }
      // 去掉年份参数
      const e = this.cronValue_c2
      // console.info('>>>>>>' + e)
      const format = 'yyyy-MM-dd hh:mm:ss'
      const options = {
        currentDate: dateFormat(this.startTime, format)
      }
      // console.info(options)
      const iter = CronParser.parseExpression(e, options)
      const result = []
      for (let i = 0; i < 5; i++) {
        result.push(dateFormat(new Date(iter.next()), format))
      }
      this.preTimeList = result.length > 0 ? result.join('\n') : '无执行时间'
    },
    calStartTime () {
      if (!this.exeStartTime) {
        this.startTime = moment(new Date(),'YYYY-MM-dd')
        return
      }
      try {
        this.startTime = moment(new Date(this.exeStartTime),'YYYY-MM-dd')
      } catch (e) {
        this.startTime = moment(new Date(),'YYYY-MM-dd')
      }
    }
  },
  components: {
    SecondUi,
    MinuteUi,
    HourUi,
    DayUi,
    WeekUi,
    MonthUi,
    YearUi
  },
  created () {
    this.formatValue()
    this.calStartTime()
    this.$nextTick(() => {
      this.calTriggerListInner()
    })
  }
}
</script>

<style scoped>
.easy-cron {
  display: inline-block;
  border: 1px solid #2d8cf0;
}

.content {
  display: flex;
  flex-wrap: nowrap;
}

.left {
  flex-basis: 60%;
  width: 60%;
  border: 1px solid transparent;
  border-right-color: #2d8cf0;
}

.right {
  flex-basis: 40%;
  width: 40%;
  /*// border: 1px solid dimgray;*/
}

.ivu-table-small td {
  height: 30px !important;
}

.exe-pre {
  margin-top: 5px;
}

.exe-pre > div {
  margin-top: 5px;
}

.exe-pre-panel {
  display: flex;
  justify-content: flex-start;
}

.exe-pre-panel .p-left {
  flex-basis: 80px;
  flex-grow: 0;
}

.exe-pre-panel .p-right {
  flex-basis: 100px;
  flex-grow: 1;
}
</style>
