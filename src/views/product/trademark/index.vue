<template>
  <div>
    <el-button
      type="primary"
      size="default"
      icon="el-icon-plus"
      @click="addTrade"
      >添加</el-button
    >
    <el-table :data="records" border style="margin: 20px 0">
      <el-table-column prop="id" label="序号" align="center" width="80">
      </el-table-column>

      <el-table-column
        prop="tmName"
        label="品牌名称"
        align="center"
        width="width"
      >
      </el-table-column>

      <el-table-column label="品牌LOGO" align="center" width="width">
        <template slot-scope="{ row }">
          <img :src="row.logoUrl" alt="logo" class="img" />
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="width">
        <template>
          <el-button type="primary" icon="el-icon-edit">修改</el-button>
          <el-button type="success" icon="el-icon-delete">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      style="text-align: center"
      @size-change="sizeChange"
      @current-change="currentChange"
      :current-page.sync="trademarkList.current"
      :page-size="trademarkList.size"
      :page-sizes="[3, 5, 7]"
      layout="total, prev, pager, next, jumper,sizes"
      :total="trademarkList.total"
      background
      :pager-count="5"
    >
    </el-pagination>
    <el-dialog title="添加品牌" :visible.sync="dialogFormVisible">
      <el-form style="width: 80%" :model="tmForm">
        <el-form-item label="品牌名称" label-width="100px">
          <el-input autocomplete="off" v-model="tmForm.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌logo" size="normal" label-width="100px">
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div class="el-upload__tip" slot="tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "Trademark",
  data() {
    return {
      page: 1,
      limit: 3,
      trademarkList: [],
      dialogFormVisible: false,
      tmForm:{
        tmName:'',
        logoUrl:''
      }
    };
  },
  mounted() {
    this.getTradeMarkList();
  },
  methods: {
    async getTradeMarkList() {
      let res = await this.$API.trademark.getPageList(this.page, this.limit);
      if (res.code === 200) {
        this.trademarkList = res.data;
      }
    },
    //每页显示数量切换
    sizeChange(limit) {
      this.limit = limit;
      this.getTradeMarkList();
    },
    //切换分页器
    currentChange(page) {
      this.page = page;
      this.getTradeMarkList();
    },
    addTrade() {
      this.dialogFormVisible = true;
      this.tmForm.tmName=''
      this.tmForm.logoUrl=''
    },
    //图片上传成功
    handleAvatarSuccess(res, file) {
          this.tmForm.logoUrl=res.data
    },
    //图片上传前
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
  },
  computed: {
    records() {
      return this.trademarkList.records;
    },
  },
};
</script>

<style>
.img {
  width: 64px;
  height: 64px;
}
.avatar-uploader .el-upload {
  border: 1px dashed #140606;
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
