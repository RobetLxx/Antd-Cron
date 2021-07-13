<template>
  <div class="show">
    <a disabled>配置项样例</a>
    <a-form :form="form">
      <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol">
            <span slot="label">
              cron表达式&nbsp;
              <a-tooltip placement="top">
                <span slot="title" style="font-size: 10px">请输入正确的cron表达式，否则会报错</span>
                <a-icon type="warning" />
              </a-tooltip>
            </span>
        <input-cron type="text" v-model="cronExpression"
                    @reset="reset" @save="oldValueSave"
                    placeholder='请输入正确的cron执行表达式'
                    v-decorator="['cronExpression',
                      {rules:[{required: true, message: 'cron执行表达式不能为空'},{validator:validateCronBlur}]}]"></input-cron>
      </a-form-item>
      <a-divider/>
      <a disabled>界面直接选择样例</a>
      <a-input disabled v-model="cronExpression1" placeholder='请在下方选定时间'></a-input>
      <easy-cron v-model="cronExpression1"></easy-cron>
    </a-form>
  </div>
</template>

<script>
// @ is an alias to /src
import EasyCron from '@/components/ecron/index'
import InputCron from '@/components/ecron/input-cron'
export default {
  name: 'Home',
  components: {
    EasyCron,
    InputCron
  },
  data(){
    return{
      oldValue:'',
      cronExpression:'',
      cronExpression1:'',
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      form: this.$form.createForm(this),
    }
  },
  methods:{
    //重置当配置中点击取消按钮则重置进入配置前的值
    reset(){
      this.cronExpression= JSON.parse(JSON.stringify(this.oldValue))
    },
    //保存
    oldValueSave(val){
      if (val==''){
        this.cronExpression= JSON.parse(JSON.stringify(this.oldValue))
        this.oldValue = '* * * * * ? *'
      }else {
        this.oldValue = val
      }
    },
    //cron校验
    validateCronBlur(rule, value, callback){
      // 没填写就不校验
      if (!value) {
        callback()
        return true
      }
      const values = value.split(' ').filter(item => !!item)
      if (values.length > 7) {
        callback('cron表达式最多7项')
        return false
      }
      // 检查第7项
      let e = value
      if (values.length === 7) {
        const year = replaceWeekName(values[6])
        if (year !== '*' && year !== '?') {
          let yearValues = []
          if (year.indexOf('-') >= 0) {
            yearValues = year.split('-')
          } else if (year.indexOf('/')) {
            yearValues = year.split('/')
          } else {
            yearValues = [year]
          }
          // console.info(yearValues)
          // 判断是否都是数字
          const checkYear = yearValues.some(item => isNaN(item))
          if (checkYear) {
            callback('cron表达式参数[年]错误:' + year)
            return false
          }
        }
        // 取其中的前六项
        e = values.slice(0, 6).join(' ')
      }
      // 6位 没有年
      // 5位没有秒、年
      let result = true
      try {
        const iter = CronParser.parseExpression(e)
        iter.next()
        callback()
      } catch (e) {
        callback('cron表达式错误,请输入正确的表达式')
        result = false
      }
      return result
    },
  }
};
</script>
<style>
.show{
  display: flex;
  width: 80%;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>
