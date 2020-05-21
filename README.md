# 涉及知识点

1. vue基础语法
2. 组件基本语法
3. 字键通讯（sync，provide，inject）
4. 插槽使用
5. prop校验
6. 过渡与动画处理
7. 计算属性与监听属性
8. v-model
9. 组件库打包
10. npm发布

# 封装组件的目的

1. 学会造轮子，了解组件库实现原理
2. 搭建和积累自己的组件库
3. 在做项目的时候缩短项目开发周期



--------------------------------------------------------------------------分割线----------------------------------------------------------------------



# 一、使用vue脚手架初始化一个项目

1.  vue created (项目名字自己定义)     例如： vue created zhengyu-ui



# 二、如何封装，注册和使用一个组件

在componet下创建一个radio.vue的文件，放置radio组件代码。，并且指定name为zyRadio。

<template>
  <label class="zy-radio"
   单选框组件
  </label>
</template>

<script>
export default {
  name: 'zyRadio'
}
</script>    

<style lang="scss">
</style>

创建组件完成后，不能在项目中直接使用，需要到main.js中注册才可以使用。

import Vue from 'vue'
import App from './App.vue'
// 第一步：导入radio组件
import radio from './components/radio.vue'

Vue.config.productionTip = false

// 第二步：注册组件,设置(组件名，组件)
Vue.component(radio.name, radio)

new Vue({
  render: h => h(App)
}).$mount('#app')



# 三、radio组件的示例代码

<template>
 <label class="zy-radio"
         :class="{'is-checked': label == model}">
    <span class="zy-radio_input">
      <span class="zy-radio_inner"></span>
      <input type="radio"
             class="zy-radio_original"
             :value="label"
             v-model="model">
    </span>
    <span class="zy-radio_label">
      <slot></slot>
      <template v-if="!$slots.default">{{label}}</template>
    </span>
  </label>
</template>

<script>
export default {
  name: 'zyRadio',
  props: {
    label: {
      type: [String, Number, Boolean],
      defualt: ''
    },
    value: null
  },
  inject: {
    RadioGroup: {
      default: ''
    }
  },
  // computed: {
  //   model: {
  //     get () {
  //       return this.isRadioGroup ? this.RadioGroup.value : this.value
  //     },
  //     set (value) {
  //       this.isRadioGroup ? this.RadioGroup.$emit('input', value) : this.$emit('input', value)
  //     }
  //   },
  //   isRadioGroup () {
  //     return !!this.RadioGroup
  //   }
  // },
  data () {
    return {
      model: this.isRadioGroup() ? this.RadioGroup.value : this.value
    }
  },
  watch: {
    value (val) {
      this.model = val
    },
    'RadioGroup.value' (val) {
      this.model = val
    },
    model (val) {
      this.isRadioGroup() ? this.RadioGroup.$emit('input', val) : this.$emit('input', val)
    }
  },
  methods: {
    isRadioGroup () {
      return !!this.RadioGroup
    }
  }
}
</script>

<style lang="scss">
.zy-radio {
  color: #606266;
  font-weight: 500;
  line-height: 1;
  position: relative;
  cursor: pointer;
  display: inline-block;
  white-space: nowrap;
  outline: none;
  font-size: 14px;
  margin-right: 30px;
  -moz-user-select: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  .zy-radio_input {
    white-space: nowrap;
    cursor: pointer;
    outline: none;
    display: inline-block;
    line-height: 1;
    position: relative;
    vertical-align: middle;
    .zy-radio_inner {
      border: 1px solid #dcdfe6;
      border-radius: 100%;
      width: 14px;
      height: 14px;
      background-color: #fff;
      position: relative;
      cursor: pointer;
      display: inline-block;
      box-sizing: border-box;
      &:after {
        width: 4px;
        height: 4px;
        border-radius: 100%;
        background-color: #fff;
        content: "";
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%) scale(0);
        transition: transform 0.15s ease-in;
      }
    }
    .zy-radio_original {
      opacity: 0;
      outline: none;
      position: absolute;
      z-index: -1;
      top: 0;
      left: 0px;
      right: 0;
      bottom: 0;
      margin: 0;
    }
  }
  .zy-radio_label {
    font-size: 14px;
    padding-left: 10px;
  }
}
// 选中的样式
.zy-radio.is-checked {
  .zy-radio_input {
    .zy-radio_inner {
      border-color: #409eff;
      background-color: #409eff;
      &:after {
        transform: translate(-50%, -50%) scale(1);
      }
    }
  }
  .zy-radio_label {
    color: #409eff;
  }
}
</style>



# 四、封装一个radio-group组件

radio-group组件是再radio组件上进行优化的，它的目的是在我们使用radio组件时，不必给每个组件都添加一个v-model，而是通过绑定一个v-model来实现数据绑定。

使用radio-group组件包裹radio组件时，我们考虑到的一个问题就是radio-group组件于radio组件之间的通讯。我们在使用radio-group组件时将数据通过v-model进行了绑定，那么raido组件就不能直接拿到这个值，所以我们需要使用provide/inject进行祖孙组件之间得传值。

使用provide/inject，在radio-group中通过声明provide对象将组件自身进行传递，在radio中使用inject进行接收即可。


# 五、radio-group组件的代码示例

<template>
      <div class="zy-radio-group">
          <slot></slot>
      </div>
</template>

<script>
export default {
  name: 'zyRadioGroup',
  provide () {
    return {
      RadioGroup: this
    }
  },
  props: {
    value: null
  }
}
</script>



# 六、封装好的组件打包成ui组件库

在项目根目录创建一个文件夹我这里是packages用于存放所有的组件 ，文件名字没有要求可以根据自己的喜好来，在packages路径下，新建一个index.js文件，用于声明install对象。 

1. 将所有的组件引入到index.js文件中
2. 声明一个数组zyComponents，将组件全部放到这个数组中
3. 定义install方法，在Vue中注册所有的组件
4. 判断是否直接引入了文件，如果引入了，则不需要调用Vue.use()方法
5. 导出install对象

<script>
// index.js示例代码
import zyRadio from './zy-radio/zy-radio.vue'
import zyRadioGroup from './zy-radio-group/zy-radio-group.vue'
const zyComponents = [
	zyRadio,
    zyRadioGroup
]
const install = function (Vue) {
  // 注册所有的组件
  zyComponents.forEach(component => {
    Vue.component(component.name, component)
  })
}
// 判断是否直接引入文件，如果是，就不用调用Vue.use()
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}
// 导出install方法
export default {
  install
}
</script>

在package.json文件中的script下加入该条指令，并且命名为lib，需要注意的是，我们需要在打包指令后面加上需要打包的路径，这里我们指定为 packages/index.js 。

打包指令打包之后文件夹名字默认dist,如果需要更改可以在--name文件名后面加上--dest 定义打包文件名

下面是示例代码：

<script>
"scripts": {
    "lib": "vue-cli-service build --target lib --name zhengyu-ui packages/index.js",
    "libs": "vue-cli-service build --target lib --name zhengyu-ui --dest lib index.js"
},
</script>



# 六、组件库上传到npm 

1. 新建一个文件夹 ，示例：zhengyu-ui
2. 把刚刚项目打包好的文件复制到文件夹里面
3. 文件夹下面新建一个 package.json 文件
4. 配置package.json  ，下面有示例代码
5. 在npm注册一个账号，有账号的可以忽略
6. 打开终端运行 npm login 登录你的npm账号
7. 终端运行 npm publish 发布到npm

<script>
{
 // 这是发布到npm的包名，发布成功之后可以直接通过npm i zhengyu-ui 下载，
 // 发布到npm上面包名一定要是npm上面没有的，否则不能上传
 "name": "zhengyu-ui",
 // 版本号
 "version": "0.1.1",
 // 如果想把包发布到npm上，我们需要将其设置位公有的包："private": false,
 "private": false,
 // 入口文件路径
 "main": "lib/zhengyu-ui.umd.min.js"
}
</script>

