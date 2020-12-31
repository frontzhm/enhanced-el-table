<template lang="pug">
div#app
  enhanced-el-table(:data="tableData",:colConfigs="colConfigs" ref="multipleTable" tooltip-effect="dark" style="width: 100%" @selection-change="handleSelectionChange")
    //- 这里的action 需要自定制列
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
