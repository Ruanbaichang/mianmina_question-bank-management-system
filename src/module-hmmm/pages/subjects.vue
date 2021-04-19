<template>
  <div class="">
    <el-card style="margin-top: 10px">
      <el-form
        :inline="true"
        class="demo-form-inline"
        :model="dataForm"
        ref="resetForm"
      >
        <el-form-item label="目录名称" prop="tagName">
          <el-input v-model="dataForm.tagName"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button @click="clearForm('resetForm')">清除</el-button>
          <el-button type="primary" @click="handleFilter">搜索</el-button>
        </el-form-item>
        <el-form-item class="right">
          <el-button
            size="small"
            style="margin-left: 10px"
            @click="addhandleChange('add')"
            type="success"
            icon="el-icon-edit"
            >新增目录</el-button
          >
        </el-form-item>

        <!-- 数据统计条条 -->
        <el-alert
          v-if="alertText !== ''"
          :title="alertText"
          type="info"
          class="alert"
          :closable="false"
          show-icon
        ></el-alert>
        <!-- 表单数据 -->
        <el-table
          :key="tableKey"
          :data="dataList"
          v-loading="listLoading"
          element-loading-text="喝口水休息一下吧"
          fit
          highlight-current-row
          style="width: 100%"
          border
        >
          <el-table-column align="center" label="序号">
            <template slot-scope="scope">
              <span>{{ scope.row.id }}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="学科名称">
            <template slot-scope="scope">
              <span>{{ scope.row.subjectName }}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="创建者">
            <template slot-scope="scope">
              <span>{{ scope.row.username }}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="创建日期">
            <template slot-scope="scope">
              <span>
                {{ scope.row.addDate | parseTimeByString("{y}-{m}-{d}") }}
              </span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="前台是否显示">
            <template slot-scope="scope">
              <span v-if="scope.row.state == 1">是</span>
              <span v-else>否</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="二级目录">
            <template slot-scope="scope">
              <span>{{ scope.row.isFrontDisplay }}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="标签">
            <template slot-scope="scope">
              <span>{{ scope.row.tags }}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="题目数量">
            <template slot-scope="scope">
              <span>{{ scope.row.twoLevelDirectory }}</span>
            </template>
          </el-table-column>
          <!-- 操作按钮 -->
          <el-table-column
            align="center"
            label="操作"
            width="280"
            class-name="small-padding fixed-width"
          >
            <template slot-scope="scope">
              <el-button
                type="text"
                size="mini"
                @click="handleStatus(scope.row)"
              >
                <span v-if="scope.row.state == 0">启用</span>
                <span v-else>禁用</span>
              </el-button>
              <el-button
                type="text"
                v-if="scope.row.status != 'deleted'"
                size="mini"
                @click="edithandleChange(scope.row.id)"
                :disabled="scope.row.state == 1 ? true : false"
              >
                编辑
              </el-button>
              <el-button
                type="text"
                v-if="scope.row.status != 'deleted'"
                size="mini"
                @click="removeTags(scope.row.id)"
                :disabled="scope.row.state == 1 ? true : false"
              >
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form>
      <!-- 分页 -->
      <div class="pagination">
        <div class="pages">
          <el-pagination
            background
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="Number(dataForm.page)"
            :total="Number(total)"
            :page-size="Number(dataForm.pagesize)"
            :page-sizes="[1, 2, 3, 10]"
            layout="sizes, prev, pager, next, jumper"
          ></el-pagination>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import { list } from '@/api/hmmm/subjects.js'
export default {
  name: '',
  components: {

  },
  data () {
    return {
      dataList: [], // 数据列表
      alertText: '', // 数据总条条
      dataForm: {
        tagName: null,
        page: 1, // 当前页
        pagesize: 10 // 页尺寸
      },
      formData: {
        tagName: ''
      },
      total: null, // 总条数
      tableKey: 0,
      listLoading: true,
      // 状态列表
      statusList: status
    }
  },
  created () {
    this.getList()
  },
  methods: {
    async getList () {
      const { data } = await list(this.dataForm)
      this.dataList = data.items
      //   console.log(data)
      this.total = data.counts
      this.alertText = `数据共 ${this.total} 条`
      this.listLoading = false
    },
    // 清空搜索框
    clearForm () {
      this.$refs.resetForm.resetFields()
    },
    // 搜索信息
    handleFilter () {
      this.dataForm.page = 1
      this.getList()
    },
    // 每页显示信息条数
    handleSizeChange (pagesize) {
      this.dataForm.pagesize = pagesize
      if (this.dataForm.page === 1) {
        this.getList()
      }
    },
    // 进入某一页
    handleCurrentChange (page) {
      this.dataForm.page = page
      this.getList()
    }
  }
}
</script>

<style lang="less" scoped>
</style>
