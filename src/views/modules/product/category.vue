<template>
  <div>
    <el-button size="mini" type="primary" @click="deleteBatch()">批量删除</el-button>
    <el-tree
      :expand-on-click-node="false"
      :data="menus"
      :props="defaultProps"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expendedKey"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="append(data)"
          >
            增加
          </el-button>
          <el-button type="text" size="mini" @click="update(data)">
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="dialogType" :visible.sync="dialogDisabled" width="30%">
      <el-form :model="category">
        <el-form-item label="分类管理">
          <el-input v-model="category.name" auto-complete="off" />
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="category.icon" auto-complete="off" />
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" auto-complete="off" />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submit()">确定</el-button>
        <el-button @click="dialogDisabled = false">取消</el-button>
      </span>
    </el-dialog>
  </div>
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
      expendedKey: [],
      dialogDisabled: false,
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: 0,
        icon: "",
        productUnit: "",
      },
      dialogType: "",
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
    append(data) {
      this.dialogType = "添加分类";
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel + 1;
      this.dialogDisabled = true;
    },
    update(data) {
      this.dialogType = "修改分类";
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        data: this.$http.adornParams(),
      }).then((info) => {
        this.category.catId = info.data.data.catId;
        this.category.name = info.data.data.name;
        this.category.icon = info.data.data.icon;
        this.category.productUnit = info.data.data.productUnit;
        this.dialogDisabled = true;
        this.getList();
      });
    },
    submit() {
      this.dialogType === "修改分类"
        ? this.updateCategory()
        : this.addCategory();
    },
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message.success("添加成功");
        this.dialogDisabled = false;
        this.getList();
        this.expendedKey = [this.category.parentCid];
      });
    },
    updateCategory() {
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        this.$message.success("修改成功");
        this.dialogDisabled = false;
        this.getList();
        this.expendedKey = [this.category.catId];
      });
    },
    remove(node, data) {
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        cancelButtonTest: "关闭",
        confirmButtomText: "确认",
        type: "warning",
      })
        .then(() => {
          var ids = [data.catId];
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(() => {
            this.$message({
              message: "删除成功",
              type: "success",
            });
            this.getList();
            this.expendedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
    deleteBatch() {
      let data=[];
      let nodeBatch = this.$refs.menuTree.getCheckedNodes();
      for(let i=0;i<nodeBatch.length;i++){
        data.push(nodeBatch[i].catId)
      }
      this.$confirm(`是否批量删除菜单?`, "提示", {
        cancelButtonTest: "关闭",
        confirmButtomText: "确认",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(data, false),
          }).then(() => {
            this.$message({
              message: "删除成功",
              type: "success",
            });
            this.getList();
          });
        }).catch(()=>{})
    },
  },
  created() {
    this.getList();
  },
};
</script>