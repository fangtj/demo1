##项目经验总结
### Vue
#####  1. Vue.set(object, key, value)
  - 改变数组某一项的值、对象增加属性
#####2. 深度watch与watch立触发回调(watch有两个可选参数)
  - deep：在选项参数中指定 deep: true，可以监听对象中属性的变化。
  - immediate： 在选项参数中指定 immediate: true, 将立即以表达式的当前值触发回调，也就是默认触发一次。
  ```
    watch:{
        //相当与组件挂载时执行一次getList方法
        data:{
            hander:'getList',
            immediate:true
        }
    }
  ```
##### 3. vue组件通信方式
  - 父子组件通信： v-bind:attr ，props
  - provide, inject
  - $parent, $children
  - ref
  - $emit, $on
  - Vuex
##### 4.  函数式组件
  - 函数式组件，即无状态，无法实例化，内部没有任何生命周期处理方法，非常轻量，
  因而渲染性能高，特别适合用来只依赖外部数据传递而变化的组件
  ```
 <template functional>
   <div>
     <p v-for="item in props.items" @click="props.itemClick(item);">
       {{ item }}
     </p>
   </div>
 </template>
   ```
##### 5. 组件复用常见bug
  -  如从/post/a到/post/b页面没有刷新，通常方法是监听$route的变化来初始化数据
  -  更优雅的解决方法：<router-view :key="$route.fullpath"></router-view>
##### 6.  打包优化提升首屏加载速度(打包后vendor.js太大)
  1. 通过启用压缩，减少打包文件体积
      - 安装compression-webpack-plugin插件， npm install --save-dev compression-webpack-plugin@1.1.2
      - 将config/index.js中的压缩属性productionGzip配置为true启用
      - webpack.base.conf.js中的output下如果有asset属性，需将属性名改为filename
  2. 将第三方插件剔除打包列表  
     将elementui、vue、echarts等剔除打包列表
      - 将elementui等js通过cdn或本地文件形式在index.html中引入，注意vue和elementui的引入顺序，vue在前，因为elementui依赖vue
      - 在webpack.base.conf.js中配置要剔除的插件
      - 在main.js及store下的index.js将vue和elementui的import语句注释掉
      ```
        //以echarts为例
        
        //index.html
        <script src="https://cdn.bootcss.com/echarts/4.2.1-rc1/echarts.min.js"></script>  
        
        //文件路径 build --> webpack.base.conf.js
        module.exports = {
             externals: { //externals 里的库不会被webpack打包
                echarts: 'echarts',
             },   
        }
      ```
##### 7. render函数
   - vue中组件只能有一个根元素，若有多个就会报错  
   
    <template>
      <li
        v-for="route in routes"
        :key="route.name"
      >
        <router-link :to="route">
          {{ route.title }}
        </router-link>
      </li>
    </template>
    
    
     ERROR - Component template should contain exactly one root element. 
        If you are using v-if on multiple elements, use v-else-if 
        to chain them instead.
   - 解决方法：使用render()函数创建HTML  
   
    functional: true,
    render(h, { props }) {
      return props.routes.map(route =>
        <li key={route.name}>
          <router-link to={route}>
            {route.title}
          </router-link>
        </li>
      )
    }
   - render函数写法
   ```
   //模板写法
   <template>
     <div class="box">
       <h2>title</h2>
       this is content
     </div>
   </template>
   
   //render函数写法
   export default {
     render (h) {
       let _c = h
       return _c('div', 
         { class: 'box'}, 
         [_c('h2', {}, 'title'), 'this is content'])
     }
   }
   ```
##### 8. 高阶组件
 - 组件的二次封装  
 假如有一个基础组件BaseList，只有基础的列表展示功能，现在我们想在这基础上增加排序功能，这个时候我们就可以创建一个高阶组件SortList
  ```
  <!-- SortList  -->
  <template>
    <BaseList v-bind="$props" v-on="$listeners"> <!-- ... --> </BaseList>
  </template>
  <script>
    import BaseList from "./BaseList";
    // 包含了基础的属性定义
    import BaseListMixin from "./BaseListMixin";
    // 封装了排序的逻辑
    import sort from "./sort.js";
   
    export default {
      props: BaseListMixin.props,
      components: {
        BaseList
      }
    };
  </script>
```
##### 9. Vue对象完整成员
    var vm = new Vue({ 
      // 数据 
      data: “声明需要响应式绑定的数据对象”, 
      props: “接收来自父组件的数据”, 
      propsData: “创建实例时手动传递props，方便测试props”, 
      computed: “计算属性”, 
      methods: “定义可以通过vm对象访问的方法”, 
      watch: “Vue实例化时会调用，
      beforeUpdate: “数据更新时调用，发生在虚拟DOM重新渲染和打补丁之前”, 
      updated: “数据更改导致虚拟DOM重新渲染和打补丁之后被调用”, 
      activated: “keep-alive组件激活时调用”, 
      deactivated: “keep-alive组件停用时调用”, 
      beforeDestroy: “实例销毁之前调用，Vue实例依然可用”, 
      destroyed: “Vue实例销毁后调用，事件监听和子实例全部被移除，释放系统资源”, 
      // 资源 
      directives: “包含Vue实例可用指令的哈希表”, 
      filters: “包含Vue实例可用过滤器的哈希表”, 
      components: “包含Vue实例可用组件的哈希表”, 
      // 组合 
      parent: “指定当前实例的父实例，子实例用this.parent访问父实例，父实例通过parent访问父实例，父实例通过children数组访问子实例”, 
      mixins: “将属性混入Vue实例对象，并在Vue自身实例对象的属性被调用之前得到执行”, 
      extends: “用于声明继承另一个组件，从而无需使用Vue.extend，便于扩展单文件组件”, 
      provide&inject: “2个属性需要一起使用，用来向所有子组件注入依赖，类似于React的Context”, 
      // 其它 
      name: “允许组件递归调用自身，便于调试时显示更加友好的警告信息”, 
      delimiters: “改变模板字符串的风格，默认为{{}}”, 
      functional: “让组件无状态(没有data)和无实例(没有this上下文)”, 
      model: “允许自定义组件使用v-model时定制prop和event”, 
      inheritAttrs: “默认情况下，父作用域的非props属性绑定会应用在子组件的根元素上。当编写嵌套有其它组件或元素的组件时，可以将该属性设置为false关闭这些默认行为”, 
      comments: “设为true时会保留并且渲染模板中的HTML注释” 
    });
   
##### 10. object.freeze()
 - 冻结一个对象，对象赋值后调用 obj = Object.freeze(obj), 冻结了obj的值，引用不会被冻结
##### 11. 监听组件生命周期
 -  `<Child @hook:mounted="doSomething"/>`  
 当然这里不仅仅是可以监听mounted，其它的生命周期事件，例如：created，updated等都可以
##### 12. 路由跳转经量使用name而不是path
- 在项目开发过程中，我们前期配置的路由路径后期难免会进行修改，
如果我们使用的是path，那么我们需要修改所有涉及到的该path的页面。
而如果我们使用的是name，即使path修改了，还可以用原来的name进行跳转
##### 13. Vue中禁止使用箭头函数的地方
-  不应该使用箭头函数来定义一个生命周期方法
-  不应该使用箭头函数来定义 method 函数
-  不应该使用箭头函数来定义计算属性函数
-  不应该对 data 属性使用箭头函数
-  不应该使用箭头函数来定义 watcher 函数
##### 14. 禁止浏览器 Auto complete 行为
```123
<el-input
  type="text"
  v-model="addData.loginName"
  readonly
  @focus="handleFocusEvent"
/>

methods: {
  handleFocusEvent(event) {
    event.target && event.target.removeAttribute('readonly')
  }
}
```
##### 15. Vue中data初始化
- 因为 props 要比 data 先完成初始化，所以我们可以利用这一点给 data 初始化一些数据进去
```
export default {
  data () {
    return {
      buttonSize: this.size
    }
  },
 props: {
   size: String
 }
}
//或者
export default {
  data (vm) {
    return {
      buttonSize: vm.size
    }
  },
 props: {
   size: String
 }
}

```
##### 16. elementUI自定义传参
  ```
  <el-autocomplete
      v-model="state4"
      :fetch-suggestions="querySearchAsync"
      placeholder="请输入内容"
      @select="((item)=>{handleSelect(item, index)})"
      // 写个闭包就可以了，index表示第几个组件
  ></el-autocomplete>
  ``` 
### css
- 设置input 文本框的 placeholder 的颜色 `input::-webkit-input-placeholder:{}`
- 设置body背景色 `html, body{height: 100%}`
- 1px边框
  ```
  .row {
    position: relative;
    &:after{
      content: "";
      position: absolute;
      left: 0;
      top: 0;
      width: 200%;
      border-bottom:1px solid #e6e6e6;
      color: red;
      height: 200%;
      -webkit-transform-origin: left top;
      transform-origin: left top;
      -webkit-transform: scale(0.5);
      transform: scale(0.5);
      pointer-events: none; /* 防止点击触发 */
      box-sizing: border-box;
    }
  }
  ```
- div实现背景颜色和背景图片同时存在
  ```
  {
      background-color: #fff;
      background-image:url('../../assets/img/model-bg.png');
      background-repeat: no-repeat;
  }
  ```
- 图片居中显示 `object-fit: cover` 会裁剪图片的一部分
- 多padding，少margin
-  position:fixed 降级问题  
  问题：父元素中有使用 transform，fixed 的效果会降级为 absolute。  
  解决：使用 fixed 的直接父元素的高度和屏幕的高度相同时 fixed 和 absolute 的表现效果会是一样的。
- 合理使用 px | em | rem | % 等单位
- 文字超出省略、文字两端对齐
  ```
  //超出省略
  .line-camp( @clamp:2 ) {
      text-overflow: -o-ellipsis-lastline;
      overflow: hidden;
      text-overflow: ellipsis;
      display: -webkit-box;
      -webkit-line-clamp: @clamp;
      /*! autoprefixer: off */
      -webkit-box-orient: vertical;
      /* autoprefixer: on */
  }
  
  //文字两端对齐
  // html
  <div>姓名</div>
  <div>手机号码</div>
  <div>账号</div>
  <div>密码</div>
  
  // css
  div {
      margin: 10px 0; 
      width: 100px;
      border: 1px solid red;
      text-align: justify;
      text-align-last:justify
  }
  div:after{
      content: '';
      display: inline-block;
      width: 100%;
  }
  ```
