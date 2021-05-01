<template>
    <el-tree
      :data="menus"
      :props="defaultProps"
      node-key="catId"
      ref="menuTree"
      @node-click="nodeClick"
    ></el-tree>
</template>
<script>
export default {
  data() {
    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      }
    };
  },
  methods: {
    getList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/category"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.categories;
      });
    },
    nodeClick(data,node,component){
        this.$emit("tree-node-click",data,node,component)
    }
  },
  created() {
    this.getList();
  },
}
</script>