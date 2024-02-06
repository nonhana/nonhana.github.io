---
title: Vue3手动实现能够拖曳边缘改变大小的容器组件
date: 2024-01-31 14:19:40
tags:
  - Vue3
  - TypeScript
  - 前端
categories:
  - [一些日常开发积累]
---

# 一、前言

最近刚刚实习，还是处于公司的一个试用实习期。因此我目前在公司干的活，几乎无一例外都是根据他们给的项目原型图，把页面一点点根据自己之前的项目经验渲染出来。自然这种单纯又基础的工作，干久了之后自然会觉得枯燥乏味，写再多的页面都是如此。

上个星期，我帮公司做一款类似于 `leetcode` 写题那样的代码编辑项目。项目只有一个主页面，是非常经典的左文章、右上代码编辑器、右下终端的经典布局，但是有一个比较新的需求，就是**文章和代码编辑器的间距是可以调整的；代码编辑器和终端的间距也是可以调整的**。

自然，面对这种需求我的第一反应是去找一款比较成熟的 Vue3 拖曳库，比如 `vue3-draggable-resizable` 之类的。但是这些库的样式无一例外的都比较不好看；并且我需要的是拖曳一整条边来实现大小的变化，无需这么多重量复杂的 API。于是，我就有了自己去封装一个可拖曳边框来实现组件大小的容器组件，在这里顺便写点东西记录一下吧，就当是自己的一些小经验的总结。

# 二、具体实现

## 拖曳事件的步骤解析

一般而言，我们对某个 div 进行边缘的拖动实现缩放的时候，可以把鼠标的步骤拆解成以下的几步（以 `vue` 为讨论背景）：

1. 鼠标移动到要被拖动的边上，该边发生一些样式变化以及鼠标指针发生样式变化来提醒用户可以进行拖动；

2. 鼠标按下，触发 `vue` 组件的 `@mousedown` 事件，此时对整个组件为了能够进行鼠标的移动监听响应并改变自身大小进行一系列的初始化设置，包括但不限于：

   1. 记录鼠标目前的位置，获取到目标 div 当前的宽度和高度；
   2. 设置函数： `doDrag` 和 `stopDrag` ，分别对应移动鼠标的时候触发的函数和松开鼠标的时候触发的函数。
   3. 设置事件监听器 `mousemove` 和 `mouseup` ，并且分别绑定上上述的两个函数。

   我们可以把上述的初始化操作直接封装成一个单独的 `initDrag` 函数，利用 `vue` 组件中的 `@mousedown` 事件传入 `$event` 来获取到鼠标的事件，进而完成上述一系列的初始化操作；

3. 鼠标保持按下状态进行移动，此时会不断的触发 `mousemove` 事件，会不断的传入鼠标事件来描述鼠标当前的运动状态信息；

4. 当 div 被放大/缩小到了合适的大小后，松开鼠标，触发在第二步中设置好的 `mouseup` 事件，把整个组件设置好的监听器都移除掉，表示当前移动结束。

## 具体的组件编写示例

了解了上述步骤之后，我们可以比较清晰的了解鼠标拖动某个 html 元素的过程。因此，我们的核心任务其实是**编写这个 `initDrag` 函数。**

接下来我给出完整的 `initDrag` 函数的代码，之后我也给出一个完整的拖曳组件 `BoxDraggable.vue` 的完整代码。

```typescript
// 初始化拖动操作的函数
function initDrag(side: string, event: MouseEvent) {
  // 记录初始鼠标位置
  startX = event.clientX;
  startY = event.clientY;

  // 获取目标元素的初始宽度和高度
  startWidth = box.value ? box.value.offsetWidth : 0;
  startHeight = box.value ? box.value.offsetHeight : 0;

  // 拖动时执行的函数
  const doDrag = (e: MouseEvent) => {
    // 计算鼠标移动的距离
    let dx = e.clientX - startX;
    let dy = e.clientY - startY;

    // 根据拖动的方向调整元素的尺寸
    if (side === "right") {
      // 向右拖动时，增加宽度
      width.value = startWidth + dx;
    }
    if (side === "left") {
      // 向左拖动时，减少宽度并调整元素的左边界位置
      width.value = startWidth - dx;
      box.value!.style.left = box.value!.offsetLeft + dx + "px";
    }
    if (side === "bottom") {
      // 向下拖动时，增加高度
      height.value = startHeight + dy;
    }
    if (side === "top") {
      // 向上拖动时，减少高度并调整元素的上边界位置
      height.value = startHeight - dy;
      box.value!.style.top = box.value!.offsetTop + dy + "px";
    }
  };

  // 结束拖动时执行的函数
  const stopDrag = () => {
    // 移除鼠标移动和鼠标释放时的事件监听
    document.removeEventListener("mousemove", doDrag);
    document.removeEventListener("mouseup", stopDrag);
  };

  // 添加事件监听器，用于响应鼠标移动和鼠标释放事件
  document.addEventListener("mousemove", doDrag);
  document.addEventListener("mouseup", stopDrag);
}
```

可以看到，我们在初始调用 `initDrag` 的时候需要传进去两个参数：side 和 event。

- side 是为了表示是那条边正在被拖曳，需要根据 side 的不同来对 box 进行不同方向的尺寸调整。
- 初始传进去的 event 仅仅只是为了获取到当前鼠标的所在位置，方便计算移动的距离。真正在移动的时候不断触发传入的鼠标事件是传到 `doDrag` 这个函数里面的。

基于以上的函数实现以及过程分析，我们不难把整个组件都实现一下，包括一些样式编写：

```vue
// BoxDraggable.vue
<template>
  <div class="draggable-box" ref="box">
    <!-- 边框用于调整大小 -->
    <div
      class="resize-handle top"
      @mousedown.prevent="initDrag('top', $event)"
    ></div>
    <div
      class="resize-handle right"
      @mousedown.prevent="initDrag('right', $event)"
    ></div>
    <div
      class="resize-handle bottom"
      @mousedown.prevent="initDrag('bottom', $event)"
    ></div>
    <div
      class="resize-handle left"
      @mousedown.prevent="initDrag('left', $event)"
    ></div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

const box = ref<HTMLElement>();
const width = ref(400);
const height = ref(400);

let startX = 0;
let startY = 0;
let startWidth = 0;
let startHeight = 0;

// 初始化拖动操作的函数
function initDrag(side: string, event: MouseEvent) {
  // 记录初始鼠标位置
  startX = event.clientX;
  startY = event.clientY;

  // 获取目标元素的初始宽度和高度
  startWidth = box.value ? box.value.offsetWidth : 0;
  startHeight = box.value ? box.value.offsetHeight : 0;

  // 拖动时执行的函数
  const doDrag = (e: MouseEvent) => {
    // 计算鼠标移动的距离
    let dx = e.clientX - startX;
    let dy = e.clientY - startY;

    // 根据拖动的方向调整元素的尺寸
    if (side === "right") {
      // 向右拖动时，增加宽度
      width.value = startWidth + dx;
    }
    if (side === "left") {
      // 向左拖动时，减少宽度并调整元素的左边界位置
      width.value = startWidth - dx;
      box.value!.style.left = box.value!.offsetLeft + dx + "px";
    }
    if (side === "bottom") {
      // 向下拖动时，增加高度
      height.value = startHeight + dy;
    }
    if (side === "top") {
      // 向上拖动时，减少高度并调整元素的上边界位置
      height.value = startHeight - dy;
      box.value!.style.top = box.value!.offsetTop + dy + "px";
    }
  };

  // 结束拖动时执行的函数
  const stopDrag = () => {
    // 移除鼠标移动和鼠标释放时的事件监听
    document.removeEventListener("mousemove", doDrag);
    document.removeEventListener("mouseup", stopDrag);
  };

  // 添加事件监听器，用于响应鼠标移动和鼠标释放事件
  document.addEventListener("mousemove", doDrag);
  document.addEventListener("mouseup", stopDrag);
}
</script>

<style scoped>
.draggable-box {
  position: relative;
  width: v-bind(width + "px");
  height: v-bind(height + "px");
  background-color: #3d3d3d;
}

.resize-handle {
  position: absolute;
  background: rgba(0, 0, 0, 0);
}

.resize-handle.top,
.resize-handle.bottom {
  left: 0;
  right: 0;
  height: 5px;
  cursor: ns-resize;
}

.resize-handle.right,
.resize-handle.left {
  top: 0;
  bottom: 0;
  width: 5px;
  cursor: ew-resize;
}

.resize-handle.top {
  top: 0;
}
.resize-handle.right {
  right: 0;
}
.resize-handle.bottom {
  bottom: 0;
}
.resize-handle.left {
  left: 0;
}

.resize-handle:hover {
  background-color: rgba(255, 255, 255, 0.5); /* 当鼠标悬停时改变背景颜色 */
}
</style>
```

这样子可以实现一个单纯的能够拖动边框，实现宽度改变的一个组件。如果想要应用这个组件，能够往标签内部传入 html 元素或者 vue 组件，还需要在四个边框的下面加一个 `<slot></slot>` 来接收传入的元素。

```vue
<template>
  <div class="draggable-box" ref="box">
    <!-- 边框用于调整大小 -->
    <div
      class="resize-handle top"
      @mousedown.prevent="initDrag('top', $event)"
    ></div>
    <div
      class="resize-handle right"
      @mousedown.prevent="initDrag('right', $event)"
    ></div>
    <div
      class="resize-handle bottom"
      @mousedown.prevent="initDrag('bottom', $event)"
    ></div>
    <div
      class="resize-handle left"
      @mousedown.prevent="initDrag('left', $event)"
    ></div>

    <!-- 默认插槽，用于插入任何传入的内容 -->
    <slot></slot>
  </div>
</template>

<!-- 其余的 <script> 和 <style> 部分保持不变 -->
```

使用时，往 `<box-draggable></box-draggable>` 标签里面传入你的内容就可以了。

```vue
<BoxDraggable>
  <div>这里是您自定义的内容</div>
  <!-- 可以插入任何有效的 HTML 或 Vue 组件 -->
</BoxDraggable>
```
