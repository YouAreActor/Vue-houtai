<template>
  <div>
    <el-button type="primary" icon="el-icon-plus" @click="visible = true"
      >添加</el-button
    >

    <el-table :data="trademarkList" border style="width: 100%; margin: 20px 0">
      <el-table-column type="index" label="序号" width="80" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称"> </el-table-column>
      <el-table-column label="品牌LOGO">
        <template slot-scope="scope">
          <!-- {{ JSON.stringify(scope) }} -->
          <!--
            scope代表所有数据
              scope.row 代表当前行所有数据
          -->
          <img class="trademark-img" :src="scope.row.logoUrl" alt="logo" />
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template>
          <el-button type="warning" icon="el-icon-edit">修改</el-button>
          <el-button type="danger" icon="el-icon-delete">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <template>
      <div class="block">
        <span class="demonstration">显示总数</span>
        <el-pagination
          class="trademark-pagination"
          @size-change="getPageList(page, $event)"
          @current-change="getPageList($event, limit)"
          :page-sizes="[3, 6, 9]"
          :page-size.sync="limit"
          :current-page="page"
          :total="total"
          layout="prev, pager, next, jumper, sizes, total"
        >
        </el-pagination>
      </div>
    </template>

    <!-- <el-dialog title="添加品牌" :visible.sync="visible" width="50%">
      <el-form
        :model="trademarkForm"
        :rules="rules"
        ref="trademarkForm"
        label-width="100px"
      >
        <el-form-item label="品牌名称" prop="tmName">
          <el-input v-model="trademarkForm.tmName"></el-input>
        </el-form-item>

        <el-form-item label="品牌LOGO" prop="logoUrl"> -->
    <!--
            前提允许跨域
              action="http://182.92.128.115/admin/product/fileUpload"
              目标服务器地址: 代理配置中 (vue.config.js)

            不允许跨域，就使用proxy
              action="/dev-api/admin/product/fileUpload"
              /dev-api -> request.js 代理
             在main.js中定义 Vue.prototype.$BASE_API = process.env.VUE_APP_BASE_API
           -->
    <!-- <el-upload
            class="avatar-uploader"
            :action="`${$BASE_API}/admin/product/fileUpload`"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img
              v-if="trademarkForm.logoUrl"
              :src="trademarkForm.logoUrl"
              class="avatar"
            />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
          <span>只能上传jpg/png文件，且不超过50kb</span>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm('trademarkForm')"
          >确 定</el-button
        >
      </span>
    </el-dialog> -->
    <!-- 添加品牌 -->
    <el-dialog title="添加品牌" :visible.sync="visible" width="50%">
      <el-form
        :model="trademarkForm"
        :rules="rules"
        ref="trademarkForm"
        label-width="100px"
      >
        <el-form-item label="品牌名称" prop="tmName">
          <el-input v-model="trademarkForm.tmName"></el-input>
        </el-form-item>
        <!--上传图片 -->
        <el-form-item label="品牌LOGO" prop="logoUrl">
          <el-upload
            class="avatar-uploader"
            :action="`${$BASE_API}/admin/product/fileUpload`"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img
              v-if="trademarkForm.logoUrl"
              :src="trademarkForm.logoUrl"
              class="avatar"
            />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
          <span>不要上传超过100Kb的图片</span>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm('trademarkForm')"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "TrademarkList",
  data() {
    return {
      trademarkList: [],
      visible: false,
      page: 1,
      limit: 3,
      total: 0,
      visible: false, // 对话框显示&隐藏
      trademarkForm: {
        //表单数据
        tmName: "",
        logoUrl: "",
      },
      rules: {
        //检验规则
        tmName: [
          {
            required: true,
            message: "请输入名称",
            tigger: "blur",
          },
        ],
        logoUrl: [
          {
            required: true,
            message: "请上传LOGO图片",
          },
        ],
      },
    };
  },

  methods: {
    //提交表单的方法
    submitForm(form) {
      this.$refs[form].validate(async (valid) => {
        if (valid) {
          //校验成功
          const result = await this.$API.trademark.addTrademark(
            this.trademarkForm
          );
          // const result = await this.$API.trademark.addTrdemark(
          //   this.trademarkForm
          // );
          if (result.code === 200) {
            this.$message.success("LOGO上传成功");
            this.visible = false;
            this.getPageList(this.page, this.limit); //重新请求
          } else {
            this.$message.error(result.message);
          }
        }
      });
    },
    //上传成功回调
    handleAvatarSuccess(res) {
      this.trademarkForm.logoUrl = res.data;
    },

    //图片上传之前的触发回调函数
    beforeAvatarUpload(file) {
      const imgTypes = ["image/jpg", "image/png", "image/jpeg"];
      //检测文件类型
      const isValidType = imgTypes.indexOf(file.type) > -1;
      //检测文件大小
      const isLt = file.size / 1024 > 50;

      if (!isValidType) {
        this.$message.error("上传头像LOGO只能是 JPG/PNG/JPEG 格式!");
      }
      if (!isLt) {
        this.$message.error("上传头像LOGO大小不能超过100kb!");
      }
      //返回true即可
      return isValidType && isLt;
    },
    //请求数据
    async getPageList(page, limit) {
      const result = await this.$API.trademark.getPageList(page, limit);
      if (result.code === 200) {
        this.$message.success("数据请求成功");
        this.trademarkList = result.data.records;
        this.page = result.data.current;
        this.limit = result.data.size;
        this.total = result.data.total;
      } else {
        this.$message.error("数据请求失败");
      }
    },
  },
  mounted() {
    this.getPageList(this.page, this.limit);
  },
};
</script>

<style lang="sass" scoped>
.trademark-img
  width: 150px

>>>.trademark-pagination
  margin-left: 50%

>>>.el-pagination__sizes
  margin-left: 250px

>>>.avatar-uploader .el-upload
  border: 1px dashed #d9d9d9
  border-radius: 6px
  cursor: pointer
  position: relative
  overflow: hidden
  &:hover
    border-color: #409EFF

>>>.avatar-uploader-icon
  font-size: 28px
  color: #8c939d
  width: 178px
  height: 178px
  line-height: 178px
  text-align: center

>>>.avatar
  width: 178px
  height: 178px
  display: block
</style>


