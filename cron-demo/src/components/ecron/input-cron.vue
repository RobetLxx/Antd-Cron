<template>
  <div class="input-cron">
    <a-input :placeholder="placeholder" v-model="editCronValue" :disabled="disabled">
      <a-icon slot="suffix" type="tool" style="color:#1890ff;" />
      <a slot="suffix" @click="showConfigDlg" class="config-btn" :disabled="disabled">
        配置
      </a>
    </a-input>
    <a-modal v-model="show" title="配置Cron表达式" :closable="false" :width="'900px'" @cancel="handleCancel" @ok="handleSave">
      <div>
        <easy-cron v-model="editCronValue" :style="`width: 100%`" :exeStartTime="exeStartTime"
                   :hideYear="hideYear" :remote="remote" :hideSecond="hideSecond"></easy-cron>
      </div>
    </a-modal>
  </div>
</template>

<script>
import EasyCron from './index'

export default {
  name: 'InputCron',
  model: {
    prop: 'cronValue',
    event: 'change'
  },
  props: {
    cronValue: {
      type: String,
      default: ''
    },
    width: {
      type: Number,
      default: 700
    },
    placeholder: {
      type: String,
      default: '请输入cron表达式'
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
      editCronValue: this.cronValue==''?'':this.cronValue,
      show: false
    }
  },
  watch: {
    cronValue (newVal, oldVal) {
      console.log('旧的值:'+oldVal)
      if (newVal === this.editCronValue) {
        return
      }
      this.editCronValue = newVal
    },
    editCronValue (newVal, oldVal) {
      this.$emit('change', newVal)
    }
  },
  methods: {
    showConfigDlg () {
      if (!this.disabled) {
        this.show = true
      }
    },
    handleCancel(){
      this.$emit('reset')
      this.show = false
    },
    handleSave(){
      this.$emit('save',this.editCronValue)
      this.show = false
    }
  },
  components: {
    EasyCron
  }
}
</script>

<style scoped>
.input-cron, .input-ui {
  /*// display: inline-block;*/
}

.input-cron .ivu-input-wrapper {
  width: 100% !important;
}

.config-btn {
  cursor: pointer;
}

.config-btn:hover {
  color: #2D8CF0;
}
</style>
