<template>
  <el-tree
    :data="menus"
    node-key="catId"
    :props="defaultProps"
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
      },
    };
  },
  methods: {
    //获取菜单
    getmenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.data;
      });
    },
    nodeClick(data,node,component){
console.log("自组建被点击:",data,node,component)
this.$emit("tree-node-click",data,node,component)
    }
  },
  created() {
    this.getmenus();
  },
};
</script>

<style>
</style>