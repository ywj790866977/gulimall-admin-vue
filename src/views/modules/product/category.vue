<template>
  <div>
    <el-tree
      :data="list"
      :props="defaultProps"
      show-checkbox
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="expandedKeys"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >Append</el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>

<script>
export default {
  data() {
    return {
      expandedKeys: [],
      list: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  created() {
    // getMenusByEs7()
    this.getMenusByEs7();
  },
  methods: {
    async getMenusByEs7() {
      const res = await this.$http({
        url: this.$http.adornUrl("/product/category/list"),
        method: "get"
      });
      this.list = res.data.data;
    },
    append(data) {
      console.log(data);
    },
    remove(node, data) {
      let ids = [data.catId];
      this.$confirm(`是否删除当前[${data.name}]菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "删除成功!"
            });
            //重新获取列表
            this.getMenusByEs7();
            //设置暂开节点
            this.expandedKeys = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
};
</script>

<style lang="scss">
</style>
