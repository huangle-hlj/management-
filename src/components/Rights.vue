<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card class="box-card">
      <template>
        <el-table stripe :data="rightList" border style="width: 100%">
          <el-table-column type="index"></el-table-column>
          <el-table-column prop="menu_name" label="名称" width="250"></el-table-column>
          <el-table-column prop="path" label="路径" width="250"></el-table-column>
          <el-table-column label="级别" width="250">
            <template slot-scope="scope">
              <el-tag type="success" v-if="scope.row.lev == 1">一级</el-tag>
              <el-tag type="danger" v-else>二级</el-tag>
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
      rightList: []
    }
  },
  created() {
    this.getPower()
  },
  methods: {
    //获取权限列表
    async getPower() {
      const { data: res } = await this.$axios.get('users/rights')
      if (!res.status) {
        this.rightList = res
      }
    }
  }
}
</script>
<style lang="less" scoped>
.box-card {
  margin-top: 10px;
}
</style>