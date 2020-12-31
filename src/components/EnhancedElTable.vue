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
    const methods = [
      "clearSelection",
      "toggleRowSelection",
      "toggleAllSelection",
      "toggleRowExpansion",
      "setCurrentRow",
      "clearSort",
      "clearFilter",
      "doLayout",
      "sort"
    ];
    methods.forEach(method => (this[method] = this.$refs.elTable[method]));
  }
};
</script>
