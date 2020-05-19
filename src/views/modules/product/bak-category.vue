<template>
  <div>
    <div class="category-header">
      <span class="header-span">拖拽</span>
      <el-switch v-model="isDraggable" active-text="开启" inactive-text="关闭">拖拽</el-switch>
      <el-button v-if="isDraggable" class="save-button" @click="batchSave">批量保存</el-button>
      <el-button type="danger" @click="batchDelete">批量删除</el-button>
    </div>

    <el-tree
      :data="list"
      :props="defaultProps"
      show-checkbox
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="expandedKeys"
      :draggable="isDraggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
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
          <el-button type="text" size="mini" @click="edit(node, data)">Edit</el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pCid: [],
      updateNodes: [],
      maxLevel: 0,
      isDraggable: false,
      dialogType: 1, //1 添加 ,2 修改
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null
      },
      dialogVisible: false,
      expandedKeys: [],
      list: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  created() {
    this.getMenusByEs7();
  },
  methods: {
    batchDelete() {
      let checkedList = this.$refs.menuTree.getCheckedNodes();
      if (checkedList.length <= 0) {
        this.$message({
          type: "error",
          message: "批量删除数据为空"
        });
        return;
      }
      let ids = checkedList.map(item => item.catId);
      // console.log(ids);
      this.$confirm(`是否删除当前[${ids}]菜单?`, "提示", {
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
              message: "菜单批量删除成功!"
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
    },
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/updates"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false)
      }).then(() => {
        this.$message({
          type: "success",
          message: "修改菜单成功!"
        });
        this.getMenusByEs7();
        this.expandedKeys = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
        // this.pCid = 0;
      });
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // console.log("tree drop: ", draggingNode, dropNode, dropType);
      //1. 父节点id
      let pCid = 0;
      let siblings = null;
      // 拖拽方式不同,获取的父id和也不同
      if (dropType == "before" || dropType == "after") {
        pCid = dropNode.parent.data.catId || 0;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.pCid.push(pCid);
      //2. 当前拖拽节点最新顺序
      for (let i = 0; i < siblings.length; i++) {
        //遍历节点如果等于当前节点,还需要添加parentCid
        if (siblings[i].data.catId == draggingNode.data.catId) {
          //修正层级
          let catLevel = draggingNode.level;
          //判断层级是否一致, 不一致需要修改
          if (siblings[i].level != draggingNode.level) {
            //修改当前节点层级
            catLevel = siblings[i].level;
            //修改子节点层级
            this.updateNodesLeve(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel
          });
        } else {
          //正常修改顺序
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
    },
    updateNodesLeve(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level
          });
          this.updateNodesLeve(node.childNodes[i]);
        }
      }
    },
    allowDrop(draggingNode, dropNode, type) {
      // console.log("allowDrop : ", draggingNode, dropNode, type);
      //总层数不能大于3
      //1.当前拖动层总层 = 最深层级 - 父层级 + 1
      this.countLevelNode(draggingNode);
      //2.当前拖动总层+父节点深度 不能大于 3
      let deep = Math.abs(this.maxLevel - draggingNode.level + 1);
      // console.log("深度: ", deep);
      //3. 判断类型
      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
      return false;
    },
    countLevelNode(node) {
      //判断子节点是否为空
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countLevelNode(node.childNodes[i]);
        }
      }
    },
    async getMenusByEs7() {
      const res = await this.$http({
        url: this.$http.adornUrl("/product/category/list"),
        method: "get"
      });
      this.list = res.data.data;
    },
    edit(node, data) {
      console.log(data);
      this.dialogVisible = true;
      this.dialogType = 2;
      this.category.name = data.name;
      this.category.catId = data.catId;
    },
    append(data) {
      this.dialogVisible = true;
      this.dialogType = 1;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    addCategory() {
      console.log(this.dialogType);
      if (this.dialogType == 1) {
        this.$http({
          url: this.$http.adornUrl("/product/category/save"),
          method: "post",
          data: this.$http.adornData(this.category, false)
        }).then(() => {
          this.dialogVisible = false;
          this.$message({
            type: "success",
            message: "添加成功!"
          });
          this.getMenusByEs7();
          this.expandedKeys = [this.category.parentCid];
          this.category.name = "";
        });
      } else if (this.dialogType == 2) {
        this.$http({
          url: this.$http.adornUrl("/product/category/update"),
          method: "post",
          data: this.$http.adornData(this.category, false)
        }).then(() => {
          this.dialogVisible = false;
          this.$message({
            type: "success",
            message: "修改成功!"
          });
          this.getMenusByEs7();
          this.expandedKeys = [this.category.parentCid];
          this.category.name = "";
        });
      }
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
.category-header {
  margin-bottom: 1rem;
  .header-span {
    width: 400;
    font-size: 1.2rem;
  }
  .save-button {
    margin-left: 2rem;
  }
}
</style>
