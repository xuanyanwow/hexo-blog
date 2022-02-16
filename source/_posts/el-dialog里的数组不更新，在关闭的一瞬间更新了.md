---
title: el-dialog里的数组不更新，在关闭的一瞬间更新了
tags: 
  - 前端
id: '297'
categories:
  - 前端
date: 2021-03-08 17:16:22
---

```markup
<el-dialog title="拆单" :visible.sync="dialog_split">
    <div v-for="(item, i) in split_goods_json" >
        <el-input v-model="split_goods_json[i].split_number" style="width: 150px;">
        </el-input>
    </div>
</el-dialog>
```

但是在方法中更新数据不会重新渲染，其实这个问题的本质也不是dialog的问题，而是vue的机制问题， https://cn.vuejs.org/v2/guide/reactivity.html

# Vue 不能检测以下数组的变动

当你利用索引直接设置一个数组项时，例如：vm.items\[indexOfItem\] = newValue 当你修改数组的长度时，例如：vm.items.length = newLength 为了解决第一类问题，以下两种方式都可以实现和 vm.items\[indexOfItem\] = newValue 相同的效果，同时也将在响应式系统内触发状态更新：

```javascript
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```