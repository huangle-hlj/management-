<template>
  <div>
    <!-- 面包导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card class="box-card">
      <!-- 栅格 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <!-- 搜索框 -->
          <el-input
            @clear="getUserInfo"
            clearable
            v-model.trim="userInfo.query"
            placeholder="请输入内容"
            class="input-with-select"
          >
            <el-button @click="getUserInfo" slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button @click="dialogVisible = true" type="primary">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表 -->
      <template>
        <el-table border :data="userList" style="width: 100%">
          <el-table-column type="index"></el-table-column>
          <el-table-column prop="user_name" label="姓名" width="180"></el-table-column>
          <el-table-column prop="user_phone" label="电话" width="180"></el-table-column>
          <el-table-column prop="user_role" label="角色"></el-table-column>
          <el-table-column label="状态">
            <template slot-scope="scope">
              <el-switch @change="changeType($event,scope.row.user_id)" v-model="scope.row.type"></el-switch>
            </template>
          </el-table-column>
          <!-- 作用域插槽的使用 -->
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-tooltip
                :enterable="false"
                class="item"
                effect="dark"
                content="修改角色"
                placement="top"
              >
                <!-- 修改用户 -->
                <el-button
                  @click="changeUser(scope.row)"
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                ></el-button>
              </el-tooltip>
              <el-button size="mini" type="danger" icon="el-icon-share"></el-button>
              <!-- 删除用户 -->
              <el-button
                @click="delUser(scope.row.user_id)"
                size="mini"
                type="success"
                icon="el-icon-delete"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </template>
      <!-- 分页 -->
      <template>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="this.userInfo.pageIndex"
          :page-sizes="[1, 2, 3, 4]"
          :page-size="this.userInfo.pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="this.total"
        ></el-pagination>
      </template>
    </el-card>
    <!-- 添加用户 -->
    <el-dialog
      @close="clearForm"
      title="添加用户"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
    >
      <el-form
        :model="ruleForm"
        :rules="rules"
        ref="ruleForm"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="ruleForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="userpass">
          <el-input v-model="ruleForm.userpass"></el-input>
        </el-form-item>
        <el-form-item label="电话号码" prop="userphone">
          <el-input v-model="ruleForm.userphone"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUserFrom">确 定</el-button>
      </span>
    </el-dialog>
    <!--  //删除用户对话框 -->
    <el-dialog title="删除用户" :visible.sync="isDelUser" width="30%">
      <span>该用户数据将永久移除，请确认操作</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isDelUser = false">取 消</el-button>
        <el-button type="primary" @click="delUserOk">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    //手机号验证
    const checkPhone = (ruele, value, cb) => {
      //手机号正则
      const regPhone = /^1[3456789]\d{9}$/
      if (regPhone.test(value)) {
        return cb()
      }
      cb(new Error('请输入正确手机号'))
    }
    return {
      //表单数据
      ruleForm: {
        username: '',
        userpass: '',
        userphone: ''
      },
      //验证规则
      rules: {
        username: [
          { required: true, message: '请输入用户名', triger: blur },
          { min: 2, max: 10, message: '请输入合理用户名', triger: blur }
        ],
        userpass: [
          { required: true, message: '请输入密码', triger: blur },
          { min: 2, max: 10, message: '请输入合理密码', triger: blur }
        ],
        userphone: [
          { required: true, message: '请输入手机号', triger: blur },
          { validator: checkPhone, triger: blur }
        ]
      },
      //控制对话框
      dialogVisible: false,
      //获取用户数据参数
      userInfo: {
        query: '',
        pageIndex: 1,
        pageSize: 2
      },
      //用户列表
      userList: [],
      //总数据
      total: 0,
      //修改用户还是add用户
      changeOrAdd: false,
      //修改的用户id
      id: '',
      //删除用户
      isDelUser: false
    }
  },
  created() {
    this.getUserInfo()
  },
  methods: {
    async getUserInfo() {
      //{data:res}解构返回值，将返回值中的data赋值给res
      const { data: res } = await this.$axios.get('users/users', {
        params: this.userInfo
      })
      this.userList = res[0]
      this.total = res[1][0].total
    },
    //监听分页数变化
    handleSizeChange(val) {
      this.userInfo.pageSize = val
      this.getUserInfo()
    },
    //监听页码变化
    handleCurrentChange(val) {
      this.userInfo.pageIndex = val
      this.getUserInfo()
    },
    //改变用户状态
    async changeType(eve, val) {
      let type
      if (eve) {
        type = 1
      } else {
        type = 0
      }
      const res = await this.$axios.put('users/users/0', {
        type,
        user_id: val
      })
      if (res.status === 201) {
        this.$message.success('状态修改成功')
      } else {
        this.$message.error('状态修改失败')
      }
    },
    //对话框关闭之前触发的事件
    handleClose() {
      console.log(1)
    },
    //关闭对话框清空内容
    clearForm() {
      this.$refs.ruleForm.resetFields()
    },
    //点击确定添加用户
    addUserFrom() {
      this.$refs.ruleForm.validate(async valid => {
        if (!valid) return
        let time = new Date().getTime()
        let ruleForm = { ...this.ruleForm, time }
        let res
        if (!this.changeOrAdd) {
          res = await this.$axios.post('users/users', ruleForm)
        } else {
          res = await this.$axios.put('users/users/' + this.id, ruleForm)
        }
        if (res.status == 201) {
          this.$message.success('操作成功')
          this.dialogVisible = false
          this.getUserInfo()
        } else {
          this.$message.error('操作失败')
        }
      })
      this.changeOrAdd = false
    },
    //修改用户
    changeUser(val) {
      //打开对话框
      this.dialogVisible = true
      //列表赋值
      this.ruleForm.username = val.user_name
      this.ruleForm.userphone = val.user_phone
      this.ruleForm.userpass = val.user_pass
      this.id = val.user_id
      this.changeOrAdd = true
    },
    //删除用户
    delUser(id) {
      //将用户id存入data
      this.id = id
      //打开对话框
      this.isDelUser = true
    },
    //确认删除用户
    async delUserOk() {
      var res = await this.$axios.delete('users/users/' + this.id)
      if (res.status == 202) {
        this.$message.success('操作成功')
        this.getUserInfo()
      } else {
        this.$message.error('操作失败')
      }
      this.isDelUser = false
    }
  }
}
</script>
<style lang="less" scoped>
.el-breadcrumb {
  margin-bottom: 10px;
}
.el-table {
  margin-top: 15px;
}
</style>