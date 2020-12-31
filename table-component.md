---
title: 让el-table更好用，通过配置的方式
tags: js
categories: js
---

`element-ui`虽然有[`el-table`组件](https://element.eleme.cn/#/zh-CN/component/table)，但是仍然需要手动写`el-table-column`。

![e-el-table1](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-table1.png)
属性 l-table`的基础上封装个`enhanced-el-table`组件。

使用的时候希望这样，不再需要手动写里面的`el-table-column`：

```html
<enhanced-el-table :data="tableData" :colConfigs="colConfigs" ></enhanced-el-table>
```

- data属性，显示的数据，`[{name:'颜酱',age:18},{...}]`
- colConfigs属性，每个`column`项的配置数组，如下

```js
colConfigs: [
    { prop: "date", label: "日期"},
    { prop: "name", label: "姓名"},
    { prop: 'address', label: '地址'}
]
```

可以结合表格看`colConfigs`就更容易理解了，其实就是将`el-table-column`上的属性换成对象的形式

![e-el-table2](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-table2.png)

本文代码后期可能较复杂，需要的话，可以去看[github代码](https://github.com/frontzhm/enhanced-el-table)

当然文末也附上了，每个小节的具体代码，有需要也可以看看

## TL;DR

- 就是写了个`enhanced-el-table`组件，充当原先的`el-table`组件
- 唯一不一样的是，额外加了一个`colConfigs`，其他属性、事件、方法同`el-table`组件
- 哦，如果表格的列项不足以用`coloConfigs`描述的话，这边提供了`slot`
- 想直接看`enhanced-el-table`组件怎么用的，包括复杂情况，直接跳到本文的`演示实例的优化`

## 组件的概况

综上，`enhanced-el-table`组件的大概就出来了。

![e-el-table4](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-table2.png)

## el-table-column的处理

实现最开始的表格：

`App.vue`里可以简单写下：

![e-el-table5](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-table5.png)

而`enhanced-el-table`内部，没有`colConfigs`的情况下，长这样

```pug
el-table(:data="data")
  el-table-column(label="日期" prop="date")
  el-table-column(label="姓名" prop="name")
  el-table-column(label="地址" prop="address")
```

当然，有了`colConfigs`，直接就循环了

```pug
el-table(:data="data")
  el-table-column(v-for="config in colConfigs"  v-bind="config" :key="config.prop")

```

## 优化

- `el-table`自身也有很多属性，这里通过简单的`v-bind="$attrs"`，将`enhanced-el-from`上面的属性自动到`el-table`。  
- 同理，`v-on="$listeners"`
- `el-table`上面的方法，稍微麻烦点，通过手动赋值
- 一般提交按钮不需要配置项，直接插入即可，这里增加`slot#footer`
- 同理，表单的开始有可能有别的描述，这里增加`slot#header`
- 部分列项，需要定制，通过`slotName`属性，表示不参与内部循环，需要自己写逻辑

```js
//   el-table(ref="elTable":data="data" v-bind="$attrs" v-on="$listeners")
  mounted() {
    const methods = [ "clearSelection", "toggleRowSelection", "toggleAllSelection", "toggleRowExpansion", "setCurrentRow", "clearSort", "clearFilter", "doLayout", "sort" ];
    methods.forEach(method => (this[method] = this.$refs.elTable[method]));
  }
```

以上逻辑将在下面的部分展示全部代码。

## 演示实例的优化

实例的效果：

![e-el-table8](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfaaca6e0d004f56a7d831623685e809~tplv-k3u1fbpfcp-zoom-1.image)

EnhancedElForm的代码如下

```vue
<template lang="pug">
  el-table(ref="elTable" :data="data" v-bind="$attrs" v-on="$listeners")
    template(v-for="colConfig in colConfigs")
      slot(v-if="colConfig.slotName" :name="colConfig.slotName" v-bind="colConfig")
      el-table-column(v-else v-bind="colConfig" :key="colConfig.prop")
</template>

<script>
export default {
  name: "enhanced-el-table",
  props: {
    data: {
      type: Array,
      default() {
        return [];
      }
    },
    colConfigs: {
      type: Array,
      default() {
        return [];
      }
    }
  },
  mounted() {
    const methods = [ "clearSelection", "toggleRowSelection", "toggleAllSelection", "toggleRowExpansion", "setCurrentRow", "clearSort", "clearFilter", "doLayout", "sort" ];
    methods.forEach(method => (this[method] = this.$refs.elTable[method]));
  }
};
</script>

```

App.vue的代码如下

```vue
<template lang="pug">
div#app
  enhanced-el-table(:data="tableData",:colConfigs="colConfigs" ref="multipleTable" tooltip-effect="dark" style="width: 100%" @selection-change="handleSelectionChange")
    template(#action="colConfig")
      el-table-column(v-bind="colConfig")
        template(#default="scope")
          el-button(size="mini" @click="handleEdit(scope.$index, scope.row)") 编辑
          el-button(type="danger" size="mini" @click="handleDelete(scope.$index, scope.row)") 删除
  div(style="margin-top: 20px")
    el-button(@click="toggleSelection([tableData[1], tableData[2]])") 切换第二、第三行的选中状态
    el-button(@click="toggleSelection()") 取消选择
  div {{multipleSelection}}
</template>

<script>
import EnhancedElTable from "./components/EnhancedElTable.vue";
export default {
  name: "App",
  components: {
    EnhancedElTable
  },
  data() {
    return {
      colConfigs: [
        { type: "selection", width: 55 },
        { prop: "date", label: "日期", width: 180 },
        { prop: "name", label: "姓名", width: 180, sortable: true },
        { prop: "address", label: "地址" },
        // 这里的action 需要自定制列
        { slotName: "action", label: "操作" }
      ],

      tableData: Array.from({ length: 6 }, (v, i) => ({
        date: `2016-5-${i + 1}`,
        name: `王小虎${i + 1}`,
        address: `上海市普陀区金沙江路 151${i + 1} 弄`
      })),

      multipleSelection: []
    };
  },
  methods: {
    toggleSelection(rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleEdit(index, row) {
      console.log(index, row);
    },
    handleDelete(index, row) {
      console.log(index, row);
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>

```

## 引用

- [优雅的使用 element-ui 中的 table 组件](https://juejin.cn/post/6844903512225087496)

## 代码

### 代码：组件的概况

```vue
<template lang="pug">
  el-table(:data="data")
</template>

<script>
export default {
  name: "enhanced-el-table",
  props: {
    data: {
      type: Array,
      default() {
        return [];
      }
    },
    colConfigs: {
      type: Array,
      default() {
        return [];
      }
    }
  }
};
</script>


```

### 代码：el-table-column的处理

App.vue

```vue

<template lang="pug">
div#app
  enhanced-el-table(:model="model" :colConfigs="colConfigs")
  div {{model}}
</template>

<script>
import EnhancedElForm from "./components/EnhancedElForm.vue";
export default {
  name: "App",
  components: { EnhancedElForm },
  data() {
    return {
      model: { name: "", age: "" },
      colConfigs: [
        {
          type: "input", modelKey: "name", label: "姓名",
          props: { maxlength: 20 },
          rules: [{ required: true, message: "请输入姓名", trigger: "blur" }]
        },
        {
          type: "input", modelKey: "age", label: "年龄",
          props: { maxlength: 5 },
          rules: [{ required: true, message: "请输入年龄", trigger: "blur" }]
        }
      ]
    };
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>

```

### 代码：除input之外的元素

EnhancedElForm.vue

```vue
<template lang="pug">
el-table(:model="model" :rules="rules")
  el-table-column(v-for="config in colConfigs" :label="config.label" :prop="config.modelKey" :key="config.modelKey")
  
    el-radio-group(v-if="config.type==='radio-group'"   v-model="model[config.modelKey]" v-bind="config.props")
      el-radio(v-for="(item,index) in config.props.options" :key="index" :label="typeof item==='object'?item.value:item") {{ typeof item==='object'?item.label:item }}
      
    el-checkbox-group(v-else-if="config.type==='checkbox-group'"   v-model="model[config.modelKey]" v-bind="config.props")
      el-checkbox(v-for="(item,index) in config.props.options" :key="index" :label="typeof item==='object'?item.value:item") {{ typeof item==='object'?item.label:item }}
      
    el-select(v-else-if="config.type==='select'"   v-model="model[config.modelKey]" v-bind="config.props")
      el-option(v-for="(item,index) in config.props.options" :key="index" :value="typeof item==='object'?item.label:item" :label="typeof item==='object'?item.value:item")

    component(v-else :is="'el-'+config.type" v-model="model[config.modelKey]" v-bind="config.props") {{config.text}}
    
</template>
<script>
export default {
  name: "enhanced-el-table",
  props: {
    model: {
      type: Object,
      default() {
        return {};
      }
    },
    colConfigs: {
      type: Array,
      default() {
        return {};
      }
    }
  },
  computed: {
    rules() {
      return this.colConfigs.reduce((acc, cur) => {
        acc[cur.modelKey] = cur.rules;
        return acc;
      }, {});
    }
  }
};
</script>

</script>

```

### 代码：演示实例 - 添加表单项