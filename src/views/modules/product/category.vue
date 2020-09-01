<template>
  <div>
    <el-tree
      show-checkbox
      :data="menus"
      node-key="catId"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      :default-expanded-keys="expendKeys"
    >
      <span
        class="custom-tree-node"
        slot-scope="{ node, data }"
      >
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          > 新增 </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)"
          > 修改 </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          > 删除</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input
            v-model="category.name"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <!-- <el-upload
            class="upload-demo"
            action="upload"
            :http-request="uploadSectionFile"
            :file-list="fileList"
            list-type="picture"
          >
            <el-button
              size="small"
              type="primary"
            >点击上传</el-button>
            <div
              slot="tip"
              class="el-upload__tip"
            >只能上传jpg/png文件，且不超过500kb</div>
          </el-upload> -->
          <el-upload
            class="avatar-uploader"
            action="upload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
            :http-request="uploadSectionFile"
          >
            <img
              v-if="imageUrl"
              :src="imageUrl"
              class="avatar"
            >
            <i
              v-else
              class="el-icon-plus avatar-uploader-icon"
            ></i>
          </el-upload>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="submitData"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>

</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      },
      title: "",
      dialogType: "",
      dialogVisible: false,
      menus: [],
      expendKeys: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      imageUrl: "",
    };
  },
  computed: {},
  watch: {},
  //方法集合
  methods: {
    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw);
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    //上传文件
    uploadSectionFile(param) {
      var fileObj = param.file;
      // FormData 对象
      var form = new FormData();
      // 文件对象
      form.append("file", fileObj);
      this.$http({
        url: this.$http.adornUrl("/upload/image/upload"),
        method: "post",
        headers: {
          "Content-Type": "multipart/form-data",
        },
        data: form,
      }).then(({ data }) => {
        console.log("图片：", data);
        this.imageUrl = data.fileUrl;
        this.category.icon = data.fileUrl;
      });
    },
    handleNodeClick(data) {},
    //获取菜单
    getmenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.data;
      });
    },

    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },

    edit(data) {
      this.title = "修改分类";
      this.dialogType = "edit";
      this.dialogVisible = true;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        this.imageUrl = data.category.icon;
        this.category.catId = data.category.catId;
        this.category.name = data.category.name;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.parentCid = data.category.parentCid;
        this.category.catLevel = data.category.catLevel;
        this.category.showStatus = data.category.showStatus;
        this.category.sort = data.category.sort;
      });
    },
    //修改分类数据
    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      var data = { catId, name, icon, productUnit };
      console.log("data:", data);
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false),
      }).then(({ data }) => {
        console.log("返回数据:",data)
        if (data.code === 0) {
          this.$message({
            type: "success",
            message: "菜单修改成功!",
          });
          this.dialogVisible = false;
          this.getmenus();
          this.expendKeys = [this.category.parentCid];
        }else{
           this.$message({
            type: "error",
            message: "菜单修改失败!",
          });
          console.log("error:",data)
        }
      });
    },

    append(data) {
      this.title = "添加分类";
      this.dialogType = "add";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.catId = null;
      this.category.showStatus = 1;
      this.category.sort = 0;
    },
    //提交分类数据
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单保存成功!",
        });
        this.dialogVisible = false;
        this.getmenus();
        this.expendKeys = [this.category.parentCid];
      });
    },

    //删除菜单
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "删除成功!",
            });
            this.getmenus();
            this.expendKeys = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
  created() {
    this.getmenus();
  },
};
</script>
<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>