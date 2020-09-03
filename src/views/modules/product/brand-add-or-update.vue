<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    @closed="dialogClosed"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="100px"
    >
      <el-form-item
        label="品牌名"
        prop="name"
      >
        <el-input
          v-model="dataForm.name"
          placeholder="品牌名"
        ></el-input>
      </el-form-item>
      <el-form-item
        label="品牌logo"
        prop="logo"
      >
        <el-upload
          class="avatar-uploader"
          action="upload"
          :show-file-list="false"
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
        <!-- <el-input v-model="dataForm.logo" placeholder="品牌logo地址"></el-input> -->
      </el-form-item>
      <el-form-item
        label="介绍"
        prop="descript"
      >
        <el-input
          v-model="dataForm.descript"
          placeholder="介绍"
        ></el-input>
      </el-form-item>
      <el-form-item
        label="显示状态"
        prop="showStatus"
      >
        <el-switch
          v-model="dataForm.showStatus"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
        ></el-switch>
      </el-form-item>
      <el-form-item
        label="检索首字母"
        prop="firstLetter"
      >
        <el-input
          v-model="dataForm.firstLetter"
          placeholder="检索首字母"
        ></el-input>
      </el-form-item>
      <el-form-item
        label="排序"
        prop="sort"
      >
        <el-input
          v-model.number="dataForm.sort"
          placeholder="排序"
        ></el-input>
      </el-form-item>
    </el-form>
    <span
      slot="footer"
      class="dialog-footer"
    >
      <el-button @click="visible = false">取消</el-button>
      <el-button
        type="primary"
        @click="dataFormSubmit()"
      >确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      imageUrl: "",
      dataForm: {
        brandId: 0,
        name: "",
        logo: "",
        descript: "",
        showStatus: "",
        firstLetter: "",
        sort: 0,
      },
      dataRule: {
        name: [{ required: true, message: "品牌名不能为空", trigger: "blur" }],
        logo: [
          { required: true, message: "品牌logo不能为空", trigger: "blur" },
        ],
        descript: [
          { required: true, message: "介绍不能为空", trigger: "blur" },
        ],
        firstLetter: [
          {
            validator: (rule, value, callback) => {
              if (value === "") {
                callback(new Error("首字母不能为空"));
              } else if (!/^[a-zA-Z]$/.test(value)) {
                callback(new Error("首字母必须a-z或者A-Z之间"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        sort: [
          {
            validator: (rule, value, callback) => {
              if (value === "") {
                callback(new Error("排序不能为空"));
              } else if (!Number.isInteger(value) || value < 0) {
                callback(new Error("排序必须是一个大于等于的整数"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
      },
    };
  },
  methods: {
     dialogClosed() {
      this.imageUrl = '';
    },
    init(id) {
      this.dataForm.brandId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.brandId) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/brand/info/${this.dataForm.brandId}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.imageUrl = data.data.logo;
              this.dataForm.name = data.data.name;
              this.dataForm.logo = data.data.logo;
              this.dataForm.descript = data.data.descript;
              this.dataForm.showStatus = data.data.showStatus;
              this.dataForm.firstLetter = data.data.firstLetter;
              this.dataForm.sort = data.data.sort;
            }
          });
        }
      });
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
        this.imageUrl = data.fileUrl;
        this.dataForm.logo = data.fileUrl;
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/brand/${!this.dataForm.brandId ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              brandId: this.dataForm.brandId || undefined,
              name: this.dataForm.name,
              logo: this.dataForm.logo,
              descript: this.dataForm.descript,
              showStatus: this.dataForm.showStatus,
              firstLetter: this.dataForm.firstLetter,
              sort: this.dataForm.sort,
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
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
  width: 160px;
  height: 160px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>