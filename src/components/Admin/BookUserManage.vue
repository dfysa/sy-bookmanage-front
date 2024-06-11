<template>
  <div class="search_container">
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>读者管理</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card shadow="always">
      <!-- 搜索内容和导出区域 -->
      <el-row>
        <el-col :span="4">
          <el-button type="primary" @click="showAddDialog()">
            <i class="el-icon-plus"></i> 添加读者
          </el-button>
        </el-col>
        <el-col :span="2" style="float: right">
          <download-excel
            class="export-excel-wrapper"
            :data="tableData"
            :fields="json_fields"
            :header="title"
            name="读者.xls"
          >
            <el-button type="primary" class="el-icon-printer" size="mini">导出Excel</el-button>
          </download-excel>
        </el-col>
        <el-col :span="2" style="float: right">
          <el-button type="primary" class="el-icon-printer" size="mini" @click="downLoad">导出PDF</el-button>
        </el-col>
        <el-col :span="2" style="float: right">
          <el-button type="success" class="el-icon-full-screen" size="mini" @click="fullScreen">全屏</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table
        :data="tableData"
        border
        style="width: 100%"
        stripe
        id="pdfDom"
        :default-sort="{ prop: 'userId', order: 'ascending' }"
        v-loading="loading"
        element-loading-text="拼命加载中"
        element-loading-spinner="el-icon-loading"
        element-loading-background="rgba(0, 0, 0, 0.8)"
      >
        <el-table-column prop="userId" label="ID" sortable></el-table-column>
        <el-table-column prop="username" label="账号"></el-table-column>
        <el-table-column prop="cardName" label="姓名"></el-table-column>
        <el-table-column prop="cardNumber" label="借阅证编号"></el-table-column>
        <el-table-column prop="ruleNumber" label="规则编号"></el-table-column>
        <el-table-column prop="status" label="状态" :formatter="formatStatus"></el-table-column>
        <el-table-column prop="gender" label="性别" :formatter="formatGender"></el-table-column>
        <el-table-column prop="age" label="年龄"></el-table-column>
        <el-table-column prop="tel" label="电话"></el-table-column>
        <el-table-column prop="address" label="住址"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-tooltip effect="dark" content="修改" placement="top" :enterable="false">
              <el-button
                type="primary"
                icon="el-icon-edit"
                size="small"
                style="height: 10px;padding: 10px 8px 15px 8px;"
                @click="showEditDialog(scope.row.userId)"
              ></el-button>
            </el-tooltip>
            <el-tooltip effect="dark" content="删除" placement="top" :enterable="false">
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="small"
                style="height: 10px;padding: 10px 8px 15px 8px;"
                @click="removeUserById(scope.row.userId)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页查询区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pageNum"
        :page-sizes="[1, 2, 3, 4, 5]"
        :page-size="queryInfo.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
      <!-- 修改读者的对话框 -->
      <el-dialog
        title="修改读者"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed"
      >
        <el-form
          :model="editForm"
          ref="editFormRef"
          :rules="editFormRules"
          label-width="100px"
        >
          <el-form-item label="姓名" prop="cardName">
            <el-input v-model="editForm.cardName"></el-input>
          </el-form-item>
          <el-form-item label="账号" prop="username">
            <el-input v-model="editForm.username"></el-input>
          </el-form-item>
 
          <el-form-item label="规则编号" prop="ruleNumber">
            <el-input v-model="editForm.ruleNumber"></el-input>
          </el-form-item>
          <el-form-item label="状态" prop="status">
   <el-select v-model="editForm.status" placeholder="请选择状态" value-key="value">
  <el-option :value="0" label="正常 "></el-option>
  <el-option :value="1" label="停用 "></el-option>
</el-select>
          </el-form-item>
          <el-form-item label="性别" prop="gender">
<el-select v-model="editForm.gender" placeholder="请选择性别" value-key="value">
  <el-option :value="0" label="男 "></el-option>
  <el-option :value="1" label="女 "></el-option>
</el-select>
          </el-form-item>
          <el-form-item label="年龄" prop="age">
            <el-input v-model="editForm.age"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="tel">
            <el-input v-model="editForm.tel"></el-input>
          </el-form-item>
          <el-form-item label="住址" prop="address">
            <el-input v-model="editForm.address"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="updateReader">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 添加读者的对话框 -->
      <el-dialog
        title="添加读者"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed"
      >
        <el-form
          :model="addForm"
          ref="addFormRef"
          :rules="addFormRules"
          label-width="100px"
        >
          <el-form-item label="姓名" prop="cardName">
            <el-input v-model="addForm.cardName"></el-input>
          </el-form-item>
          <el-form-item label="账号" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="借阅证编号" prop="cardNumber">
            <el-input v-model="addForm.cardNumber"></el-input>
          </el-form-item>
          <el-form-item label="规则编号" prop="ruleNumber">
            <el-input v-model="addForm.ruleNumber"></el-input>
          </el-form-item>
      <el-form-item label="状态" prop="status">
<el-select v-model="addForm.status" placeholder="请选择状态" value-key="value">
  <el-option :value="0" label="正常 "></el-option>
  <el-option :value="1" label="停用 "></el-option>
</el-select>
</el-form-item>
          <el-form-item label="性别" prop="gender">
<el-select v-model="addForm.gender" placeholder="请选择性别" value-key="value">
  <el-option :value="0" label="男 "></el-option>
  <el-option :value="1" label="女 "></el-option>
</el-select>
          </el-form-item>
          <el-form-item label="年龄" prop="age">
            <el-input v-model="addForm.age"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="tel">
            <el-input v-model="addForm.tel"></el-input>
          </el-form-item>
          <el-form-item label="住址" prop="address">
            <el-input v-model="addForm.address"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addReader">添加读者</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableData: [],
      editDialogVisible: false,
      editForm: {
        cardName: "",
        username: "",
        cardNumber: "",
        ruleNumber: "",
        status: 0,
        gender: 0,
        age: "",
        tel: "",
        address: ""
      },
      editFormRules: {
        cardName: [
          { required: true, message: "请输入姓名", trigger: "blur" }
        ],
        username: [
          { required: true, message: "请输入账号", trigger: "blur" },
          {
            min: 5,
            max: 30,
            message: "长度在5到30个字符",
            trigger: "blur",
          },
        ],
        
        ruleNumber: [
          { required: true, message: "请输入规则编号", trigger: "blur" }
        ],
        status: [
          { required: true, message: "请选择状态", trigger: "change" }
        ],
        gender: [
          { required: true, message: "请输入性别", trigger: "change" }
        ],
        age: [
          { required: true, message: "请输入年龄", trigger: "blur" }
        ],
        tel: [
          { required: true, message: "请输入电话", trigger: "blur" }
        ],
        address: [
          { required: true, message: "请输入住址", trigger: "blur" }
        ],
      },
      addDialogVisible: false,
      addForm: {
        cardName: "",
        username: "",
        cardNumber: "",
        ruleNumber: "",
        status: 0,
        gender: 0,
        age: "",
        tel: "",
        address: ""
      },
      addFormRules: {
        cardName: [
          { required: true, message: "请输入姓名", trigger: "blur" }
        ],
        username: [
          { required: true, message: "请输入账号", trigger: "blur" },
          {
            min: 5,
            max: 30,
            message: "长度在5到30个字符",
            trigger: "blur",
          },
        ],
        cardNumber: [
          { required: true, message: "请输入借阅证编号", trigger: "blur" }
        ],
        ruleNumber: [
          { required: true, message: "请输入规则编号", trigger: "blur" }
        ],
        status: [
          { required: true, message: "请选择状态", trigger: "change" }
        ],
        gender: [
          { required: true, message: "请输入性别", trigger: "change" }
        ],
        age: [
          { required: true, message: "请输入年龄", trigger: "blur" }
        ],
        tel: [
          { required: true, message: "请输入电话", trigger: "blur" }
        ],
        address: [
          { required: true, message: "请输入住址", trigger: "blur" }
        ],
      },
      queryInfo: {
        pageNum: 1,
        pageSize: 5,
      },
      total: 0,
      title: "读者",
      json_fields: {
        读者ID: "userId",
        账号: "username",
        姓名: "cardName",
        借阅证编号: "cardNumber",
        规则编号: "ruleNumber",
        状态: "status",
        性别: "gender",
        年龄: "age",
        电话: "tel",
        住址: "address"
      },
      loading: true
    };
  },
  methods: {
      formatStatus(row) {
      return row.status === 0 ? '正常' : '停用';
    },
        formatGender(row) {
      return row.status === 0 ? '男' : '女';
    },
    handleSizeChange(val) {
      this.queryInfo.pageSize = val;
      this.getReaderList();
    },
    handleCurrentChange(val) {
      this.queryInfo.pageNum = val;
      this.getReaderList();
    },
    async showEditDialog(id) {
      const { data: res } = await this.$http.get("admin/get/" + id);
      if (res.status !== 200) {
        return this.$message.error(res.msg);
      }
      this.editForm = res.data;
      this.editDialogVisible = true;
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    async removeUserById(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该读者信息, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((error) => {
        return error;
      });
      if (confirmResult !== "confirm") {
        return this.$message.info("已经取消删除");
      }
      const { data: res } = await this.$http.delete(
        "admin/delete/" + id
      );
      if (res.status !== 200) {
        return this.$message.error(res.msg);
      }
      this.$message.success({
        message: res.msg,
        duration: 1000,
      });
      this.getReaderList();
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    showAddDialog() {
      this.addDialogVisible = true;
    },
    async getReaderList() {
      this.loading = true;
      const { data: res } = await this.$http.post(
        "admin/list",
        this.queryInfo
      );
      if (res.status !== 200) {
        this.loading = false;
        return this.$message.error(res.msg);
      }
      this.$message.success({
        message: res.msg,
        duration: 1000,
      });
      this.tableData = res.data.records;
      this.total = parseInt(res.data.total);
      this.loading = false;
    },
    async addReader() {
      const { data: res } = await this.$http.post(
        "admin/add",
        this.addForm
      );
      if (res.status !== 200) {
        return this.$message.error(res.msg);
      }
      this.$message.success({
        message: res.msg,
        duration: 1500,
      });
      this.getReaderList();
      this.addDialogVisible = false;
    },
    async updateReader() {
      const { data: res } = await this.$http.put(
        "admin/update",
        this.editForm
      );
      if (res.status !== 200) {
        return this.$message.error(res.msg);
      }
      this.$message.success({
        message: res.msg,
        duration: 1500,
      });
      this.getReaderList();
      this.editDialogVisible = false;
    },
    downLoad() {
      this.getPdf(this.title); //参数是下载的pdf文件名
    },
    fullScreen() {
      let full = document.fullscreenElement;
      if (!full) {
        document.documentElement.requestFullscreen();
      } else {
        document.exitFullscreen();
      }
    }
  },
  created() {
    this.getReaderList();
  },
};

</script>

<style>
/* .el-button--mini {
 font-size: 1px !important;
  padding: 5px 1px !important;
} */
</style>