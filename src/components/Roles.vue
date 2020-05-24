<template>
  <div>
    <!-- 面包导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <!-- 按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 内容区域 -->
      <template>
        <el-table stripe :data="rolelist" border style="width: 100%">
          <!-- 展开列 -->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-row
                :class="['bottom',index === 0?'top':'']"
                v-for="(item,index) in scope.row.role_type"
                :key="item.role_id"
              >
                <!-- 一级权限 -->
                <el-col :span="5">
                  <el-tag>{{item.name}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!-- 二级权限 -->
                <el-col :span="5">
                  <el-row v-for="item1 in item.children" :key="item1.id">
                    <el-col>
                      <el-tag
                        @close="delType(scope.row.role_id,item1.id)"
                        closable
                        type="warning"
                      >{{item1.name}}</el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <!-- 索引列 -->
          <el-table-column type="index"></el-table-column>
          <el-table-column prop="role_name" label="名称"></el-table-column>
          <el-table-column prop="role_detail" label="描述"></el-table-column>
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
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rolelist: []
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    //获取角色列表
    async getRoleList() {
      var res = await this.$axios.get('users/roles')
      if (res.status === 200) {
        this.rolelist = res.data
      }
    },
    //删除角色权限
    delType(id1, id2) {
      this.$messageBox
        .confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(() => {
          const roletype = [...this.rolelist][id1 - 1].role_type
          console.log(roletype)
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    }
  }
}
</script>
<style lang="less" scoped>
.box-card {
  margin-top: 10px;
}
.el-tag {
  margin: 7px;
}
.top {
  border-top: 1px solid #eee;
}
.bottom {
  border-bottom: 1px solid #eee;
}
.el-row {
  display: flex;
  align-items: center;
}
</style>