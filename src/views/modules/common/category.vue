<template>
  <div>
    <el-tree :data="list" :props="defaultProps" node-key="catId" ref="menuTree"></el-tree>
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