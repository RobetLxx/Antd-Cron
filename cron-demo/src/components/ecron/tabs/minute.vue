<template>
  <div class="config-list">
    <a-radio-group v-model="type">
      <div class="item">
        <a-radio value="TYPE_EVERY" class="choice" :disabled="disabled">每分</a-radio>
      </div>
      <div class="item">
        <a-radio value="TYPE_RANGE" class="choice" :disabled="disabled">区间</a-radio>
        从<a-input-number :disabled="type!=TYPE_RANGE || disabled" :max="maxValue" :min="minValue" :precision="0"
                         class="w60" v-model="valueRange.start" />分
        至<a-input-number :disabled="type!=TYPE_RANGE || disabled" :max="maxValue" :min="minValue" :precision="0"
                         class="w60" v-model="valueRange.end" />分
      </div>
      <div class="item">
        <a-radio value="TYPE_LOOP" class="choice" :disabled="disabled">循环</a-radio>
        从<a-input-number :disabled="type!=TYPE_LOOP || disabled" :max="maxValue" :min="minValue" :precision="0"
                         class="w60" v-model="valueLoop.start" />分开始，间隔
        <a-input-number :disabled="type!=TYPE_LOOP || disabled" :max="maxValue" :min="minValue" :precision="0"
                        class="w60" v-model="valueLoop.interval" />分
      </div>
      <div class="item">
        <a-radio value="TYPE_SPECIFY" class="choice" :disabled="disabled">指定</a-radio>
        <div class="list">
          <a-checkbox-group v-model="valueList">
            <a-checkbox class="list-check-item" v-for="i in maxValue+1"
                        :value="i-1" :key="`key-${i-1}`" :disabled="type!=TYPE_SPECIFY || disabled">
              {{i-1}}
            </a-checkbox>
          </a-checkbox-group>
        </div>
      </div>
    </a-radio-group>
  </div>
</template>

<script>
import mixin from './mixin'

export default {
  name: 'minute',
  mixins: [mixin],
  data () {
    return {}
  },
  watch: {
    value_c (newVal, oldVal) {
      this.$emit('change', newVal)
    }
  },
  created () {
    this.DEFAULT_VALUE = '*'
    this.minValue = 0
    this.maxValue = 59
    this.valueRange.start = 0
    this.valueRange.end = 59
    this.valueLoop.start = 0
    this.valueLoop.interval = 1
    this.parseProp(this.prop)
  }
}
</script>

<style scoped>

.config-list {
  text-align: left;
  margin: 0 10px 10px 10px;
}

.item {
  margin-top: 5px;
}

.choice {
  border: 1px solid transparent;
  padding: 5px 8px;
}

.choice:hover {
  border: 1px solid #2d8cf0;
}

.w60 {
  width: 60px;
}

.ivu-input-number {
  margin-left: 5px;
  margin-right: 5px;
}

.list {
  margin: 0 20px;
}

.list-check-item {
  /*padding: 1px 3px;*/
  width: 4em;
}
</style>
