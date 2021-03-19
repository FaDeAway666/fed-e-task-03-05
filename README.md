# 1、Vue 3.0 性能提升主要是通过哪几方面体现的？

响应式系统升级

- Vue2.x的响应式核心为defineProperty
- Vue3.0使用Proxy重写响应式系统
  - 可监听动态新增的属性
  - 可监听属性的删除
  - 可监听数组的索引和length属性，修改数组某一项值，现在也可被监听

编译优化

- Vue2.x的编译过程：

  template编译成render函数时，通过标记静态根节点来优化diff操作

- Vue3.0标记和提升所有静态根节点，diff时只需要对比动态节点内容

  - 引入Fragments，模板中不再需要创建一个唯一的根节点
  - 静态内容提升：组件中会把静态的内容单独提出去，只在初始化的时候创建，在render的时候将不再创建
  - Patch Flag：编译的时候会把动态节点打上标记，diff的时候只会比较标记的部分是否会改变，比如标记为TEXT，就只会比较文本节点是否有变化
  - 缓存事件处理函数，减少不必要的更新操作

优化打包体积

- 移除了一些不常用的API，例如inline-template、filter
- 对Tree-shaking的支持度更高

# 2、Vue 3.0 所采用的 Composition Api 与 Vue 2.x使用的Options Api 有什么区别？

- Options API
  - 包含一个描述组件选项（data、methods、props等）的对象
  - Options API开发复杂组件，同一个功能逻辑的代码可能被拆分到不同选项，就是一个功能需要在多个选项中编写代码，且复用时较为不便（即使有mixin，也容易引起命名冲突）
- Composition API
  - 一组基于函数的API
  - 可以更灵活地组织组件的逻辑，当查看或者修改某个功能的时候，只需要关注一块区域即可，也方便其他组件重用

# 3、Proxy 相对于 Object.defineProperty 有哪些优点？

- Proxy为对象创建一个代理，可以监听对象的多种操作（get/set/delete/defineProperty/has...）并进行相应拦截，而Object.defineProperty只能监听get/set
- Proxy是非侵入式的，返回一个新的对象，而不用对原对象进行操作
- Proxy对于数组对象的监听更为完善，能监听到length、array[index]对应元素的变化

# 4、Vue 3.0 在编译方面有哪些优化？

Vue3.0标记和提升所有静态根节点，diff时只需要对比动态节点内容

- 引入Fragments，模板中不再需要创建一个唯一的根节点
- 静态内容提升：组件中会把静态的内容单独提出去，只在初始化的时候创建，在render的时候将不再创建
- Patch Flag：编译的时候会把动态节点打上标记，diff的时候只会比较标记的部分是否会改变，比如标记为TEXT，就只会比较文本节点是否有变化
- 缓存事件处理函数，减少不必要的更新操作

# 5、Vue.js 3.0 响应式系统的实现原理？

Vue3.0响应式的过程为：

1. 组件初始化
2. 创建effect作为组件渲染Watcher
3. 使用reactive处理data
4. 对data进行依赖收集
5. 当响应式对象更新的时候触发effect更新视图

effect的创建过程：

- 创建组件，构建响应式，得到Observer对象
- 创建一个渲染effect函数，添加effect初始化属性
- 立即执行effect当中的回调，并将回调赋值给activeEffect，并放入effectStack当中

依赖收集的过程：

依赖收集发生在数据访问（get）的阶段，总共可分以下几个步骤：

1. 判断处理的数据是否为对象或数组，如果是基础类型直接返回，如果是响应式对象，将其原封不动返回

2. 对特殊的key进行处理

3. 判断对象是否为数组，如果是数组，使用arrayInstrumentations去处理数组的依赖收集

4. 通过Reflect.get进行get操作，并执行track函数收集依赖

   依赖的收集其实就是收集**用户需要在依赖变化后执行的副作用函数**

   假设target作为响应式处理的原始数据，key作为访问的属性

   首先创建一个全局的targetMap作为原始数据的map（这是一个WeakMap），targetMap的键是target，值是depsMap

   depsMap则是依赖的map，键是target的key，值是depSet，depSet是依赖（副作用函数）的集合

   将当前激活的副作用函数添加到集合当中

5. 对get的结果值进行处理，如果为数组或对象，则递归调用reactive进行响应式处理

派发更新的过程：

派发更新发生在数据更新（set）阶段，首先通过Reflect.set进行数据更新，在执行trigger函数派发更新

trigger函数的执行流程：

- 根据target 和 key从targetMap开始查找，根据target找到当前target对应的depsMap，再根据key找到这个key对应的depSet
- 遍历depSet，执行集合中每一个回调