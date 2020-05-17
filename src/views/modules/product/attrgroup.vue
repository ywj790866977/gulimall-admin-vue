<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="6">
        <category></category>
      </el-col>
      <el-col :span="18">表格</el-col>
    </el-row>
  </div>
  <!-- <div>1111</div> -->
</template>

<script>
import Category from "../common/category";
export default {
  components: { Category },
  data() {
    return {};
  },
  created() {},

  methods: {}
};
</script>

<style>
</style>