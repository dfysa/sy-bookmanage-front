<template>
  <div class="information_container">
    <div class="backgroundImg">
      <img
        src="https://xxx.xiaobaitiao.icu/img/icu/202312211243635.jpg"
        alt="背景图片"
      />
    </div>
    <div class="information_header">
      <p>个人信息</p>
      <p>
        <i class="el-icon-s-flag"></i> By reading we enrich the mind, by
        conversation we polish it.
      </p>
    </div>
    <div class="information_banner"
    v-loading="loading"
        element-loading-text="拼命加载中"
    element-loading-spinner="el-icon-loading"
    element-loading-background="rgba(0, 0, 0, 0.8)">
      <div class="information_banner_left" >
        <div class="banner_left_main" v-if="show">
          <div class="number">
            <i class="el-icon-collection-tag"></i> 借阅证编号:
            {{ this.user.cardNumber }}
          </div>
          <div class="name">
            <i class="iconfont icon-gerenxinxi"></i> 借阅证姓名:
            {{ this.user.cardName }}
          </div>
           <div class="rule">
            年龄: {{ user.age }}
          </div>
          <div class="rule">
            性别: {{ user.gender }}
          </div>
          <div class="rule">
            电话: {{ user.tel }}
          </div>
          <div class="rule">
            地址: {{ user.address }}
          </div>
          <div class="rule">
            <i class="iconfont icon-guizeshezhi"></i> 规则编号:
            {{ this.user.ruleNumber }}
          </div>
          <div class="status">
            <i class="el-icon-refresh"></i> 状态:
            {{ this.user.status === 1 ? "可用" : "禁用" }}
          </div>
        </div>
      </div>
      <div class="information_banner_right">
        <el-button type="primary" class="changePWD" @click="showEditDialog"  v-if="show"
          >修改密码</el-button
        >
        <el-button type="primary" class="changeInfo" @click="showEditInfoDialog" v-if="show">
          修改个人信息
        </el-button>
      </div>
      <el-dialog
        title="修改密码"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed"
      >
        <el-form
          :model="editForm"
          ref="editFormRef"
          :rules="editFormRules"
          label-width="120px"
        >
          <el-form-item label="新密码" prop="password">
            <el-input v-model="editForm.password" type="password" placeholder="请输入新密码"></el-input>
          </el-form-item>
          <el-form-item label="确认密码" prop="confirmPassword">
            <el-input v-model="editForm.confirmPassword" type="password" placeholder="请再次输入新密码"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="changePassword"
            >确 定</el-button
          >
        </span>
      </el-dialog>

       <el-dialog
        title="修改个人信息"
        :visible.sync="editInfoDialogVisible"
        width="50%"
        @close="editInfoDialogClosed"
      >
        <el-form
          :model="editInfoForm"
          ref="editInfoFormRef"
          label-width="120px"
        >
          <el-form-item label="年龄">
            <el-input v-model="editInfoForm.age"></el-input>
          </el-form-item>
          <el-form-item label="性别">
            <el-input v-model="editInfoForm.gender"></el-input>
          </el-form-item>
          <el-form-item label="电话">
            <el-input v-model="editInfoForm.tel"></el-input>
          </el-form-item>
          <el-form-item label="地址">
            <el-input v-model="editInfoForm.address"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editInfoDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="updateUserInfo">确 定</el-button>
        </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    var validatePass2 = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'));
      } else if (value !== this.editForm.password) {
        callback(new Error('两次输入密码不一致!'));
      } else {
        callback();
      }
    };
    return {
      user: {
        ruleNumber: Number,
        cardNumber: Number,
        status: Number,
        userId: Number,
        cardName: "",
        username: "",
        password: "",
        createTime: "",
        updateTime: "",
      },
      editForm: {
        password: "",
        confirmPassword: "",
      },
      editFormRules: {
        password: [
          { required: true, message: "请输入新密码", trigger: "blur" },
          { min: 6, max: 15, message: "新密码长度在6-15个字符", trigger: "blur" },
        ],
        confirmPassword: [
          { validator: validatePass2, trigger: "blur" }
        ],
      },
      editInfoForm: {
        age: "",
        gender: "",
        tel: "",
        address: "",
      },
      editDialogVisible: false,
      editInfoDialogVisible: false,
      show: false,
      loading: true,
    };
  },
  methods: {
    showEditDialog() {
      this.editDialogVisible = true;
    },
    showEditInfoDialog() {
      this.editInfoForm = {
        age: this.user.age,
        gender: this.user.gender,
        tel: this.user.tel,
        address: this.user.address,
      };
      this.editInfoDialogVisible = true;
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    editInfoDialogClosed() {
      this.$refs.editInfoFormRef.resetFields();
    },
    async getUserInformaton() {
      const userId = window.sessionStorage.getItem("userId");
      this.loading = true;
      const { data: res } = await this.$http.get("user/get_information/" + userId);
      if (res.status !== 200) {
        return this.$message.error(res.msg);
      }
      this.$message.success({ message: res.msg, duration: 1000 });
      this.user = res.data;
      this.show = true;
      this.loading = false;
    },
    async changePassword() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (valid) {
          const { data: res } = await this.$http.post("user/update_password", {
            password: this.editForm.password,
            userId: window.sessionStorage.getItem("userId"),
          });
          if (res.status !== 200) {
            return this.$message.error(res.msg);
          }
          this.$message.success(res.msg);
          this.editDialogVisible = false;
          this.$refs.editFormRef.resetFields();
          window.sessionStorage.clear();
          this.$router.push("/login");
        } else {
          this.$message.error("请正确填写表单");
        }
      });
    },
    async updateUserInfo() {
      const { data: res } = await this.$http.post("user/edit", {
        age: this.editInfoForm.age,
        gender: this.editInfoForm.gender,
        tel: this.editInfoForm.tel,
        address: this.editInfoForm.address,
        userId: window.sessionStorage.getItem("userId"),
      });
      if (res.status !== 200) {
        return this.$message.error(res.msg);
      }
      this.$message.success(res.msg);
      this.editInfoDialogVisible = false;
      this.getUserInformaton();
    },
  },
  created() {
    this.getUserInformaton();
  },
};
</script>

<style lang="less" scoped>
.information_container {
  position: relative;
  height: 100%;
}
.backgroundImg {
  position: absolute;
  width: 100%;
  height: 100%;
  img {
    width: 100%;
    height: 100%;
    opacity: 0.3;
  }
}
.information_header {
  height: 100px;
  text-align: center;
  p:nth-child(1) {
    line-height: 140px;
    color: black;
    font-size: 36px;
  }
  p:nth-child(2) {
    color: black;
    font-size: 24px;
  }
}
.information_banner {
  display: flex;
  flex-direction: column;
  height: 400px;
  .information_banner_left {
    flex: 0.5;
    text-align: center;
  }
  .information_banner_right {
    flex: 0.5;
    text-align: center;
    line-height: 400px;
  }
}
.banner_left_main {
  margin-top: 120px;
  color: black;
  font-size: 20px;
  div {
    margin-top: 15px;
  }
}
.changePWD {
  position: absolute;
  top: 90%;
  left: 37%;
}
.changeInfo {
  position: absolute;
  top: 90%;
  left: 53%;
}
</style>
