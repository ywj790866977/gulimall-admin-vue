<template>
  <div>
    <el-tree
      :data="list"
      :props="defaultProps"
      node-key="catId"
      ref="menuTree"
      @node-click="nodeClick"
    ></el-tree>
  </div>
</template>

<script>
export default {
  data() {
    return {
      list: [],
      expandedKeys: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  created() {
    this.getMenus();
  },
  methods: {
    nodeClick(data, node, component) {
      this.$emit("tree-node-click", data, node, component);
    },
    async getMenus() {
      const res = await this.$http({
        url: this.$http.adornUrl("/product/category/list"),
        method: "get"
      });
      this.list = res.data.data;
    }
  }
};
</script>

<style>
</style>