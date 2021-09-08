# Antd-Cron
antd框架下cron表达式可视化组件

# demo展示
![iShot2021-09-08 17](https://user-images.githubusercontent.com/38268592/132485400-66700666-6232-4b87-8d0c-74e3edbf7af3.gif)

# 使用方法
1、复制components中的文件到自己的组件文件夹下；

2、全局引入 - main.js中引入刚刚复制到components文件夹的index或者input-cron
```vue
import EasyCron from '@/components/ecron/index'

import InputCron from '@/components/ecron/input-cron'

Vue.use(EasyCron)

Vue.use(InputCron)
```


3、按需引入 - 在需要用到的界面引入,并设置component
```vue
import EasyCron from '@/components/ecron/index'

import InputCron from '@/components/ecron/input-cron'

components: {
  EasyCron,
  InputCron
},
```

4、自定义化改造

可以根据自己的项目需求去调整样式，这都是支持的
