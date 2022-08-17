title: Vue3 动画 进入过渡 & 离开过渡
date: 2022-06-07 09:32:00
toc: false
index_img: https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rPqFqGEyYkuRdiaG2TiaYiaZZYzTR5hq7tVRw/0
category:
- 前端
tags:
- 自动
- 不同
- vue
- vue3
- 动画
- 过渡
- 进入
- 离开
- JavaScript
---

> 原文：[进入过渡 & 离开过渡 | Vue.js](https://v3.cn.vuejs.org/guide/transitions-enterleave.html#%E8%BF%87%E6%B8%A1-class)，本站仅作保存记录及调整排版，不做盈利性质的商业行为。

# [#](https://v3.cn.vuejs.org/guide/transitions-enterleave.html#%E8%BF%9B%E5%85%A5%E8%BF%87%E6%B8%A1-%E7%A6%BB%E5%BC%80%E8%BF%87%E6%B8%A1)进入过渡 & 离开过渡

在插入、更新或从 DOM 中移除项时，Vue 提供了多种应用转换效果的方法。这包括以下工具：

* 自动为 CSS 过渡和动画应用 class；
* 集成第三方 CSS 动画库，例如 [animate.css](https://animate.style/) ；
* 在过渡钩子期间使用 JavaScript 直接操作 DOM；
* 集成第三方 JavaScript 动画库。

在这里，我们只介绍进入、离开的过渡，你也可以从下一节中学习[列表过渡](https://v3.cn.vuejs.org/guide/transitions-list.html)和[管理过渡状态](https://v3.cn.vuejs.org/guide/transitions-state.html) 。

## [#](https://v3.cn.vuejs.org/guide/transitions-enterleave.html#%E5%8D%95%E5%85%83%E7%B4%A0-%E7%BB%84%E4%BB%B6%E7%9A%84%E8%BF%87%E6%B8%A1)单元素/组件的过渡

Vue 提供了 `transition` 的封装组件，在下列情形中，可以给任何元素和组件添加进入/离开过渡

* 条件渲染 (使用 `v-if`)
* 条件展示 (使用 `v-show`)
* 动态组件
* 组件根节点

这里是一个典型的例子：

```html
<div id="demo">
  <button @click="show = !show">
    Toggle
  </button>

  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>
```

```js
const Demo = {
  data() {
    return {
      show: true
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

```css
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

当插入或删除包含在 `transition` 组件中的元素时，Vue 将会做以下处理：

1. 自动嗅探目标元素是否应用了 CSS 过渡或动画，如果是，在恰当的时机添加/删除 CSS 类名。
2. 如果过渡组件提供了 [JavaScript 钩子函数](https://v3.cn.vuejs.org/guide/transitions-enterleave.html#javascript-%E9%92%A9%E5%AD%90) ，这些钩子函数将在恰当的时机被调用。
3. 如果没有找到 JavaScript 钩子并且也没有检测到 CSS 过渡/动画，DOM 操作 (插入/删除) 在下一帧中立即执行。(注意：此处指浏览器逐帧动画机制，和 Vue 的 `nextTick` 概念不同)

### [#](https://v3.cn.vuejs.org/guide/transitions-enterleave.html#%E8%BF%87%E6%B8%A1-class)过渡 class

在进入/离开的过渡中，会有 6 个 class 切换。

1. `v-enter-from`：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。
2. `v-enter-active`：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。
3. `v-enter-to`：定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 `v-enter-from` 被移除)，在过渡/动画完成之后移除。
4. `v-leave-from`：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
5. `v-leave-active`：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。
6. `v-leave-to`：离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 `v-leave-from` 被移除)，在过渡/动画完成之后移除。

![Transition Diagram](https://www.biuaxia.cn/usr/uploads/2022/06/3724706116.svg)

这里的每个 class 都将以过渡的名字添加前缀。如果你使用了一个没有名字的 `<transition>`，则 `v-` 是这些 class 名的默认前缀。举例来说，如果你使用了 `<transition name="my-transition">`，那么 `v-enter-from` 会替换为 `my-transition-enter-from`。

`v-enter-active` 和 `v-leave-active` 可以控制进入/离开过渡的不同的缓和曲线，在下面章节会有个示例说明。
