<template>
  <div class="container">
    <!-- 顶部表单区域 -->
    <el-card style="margin-top: 10px">
      <el-breadcrumb separator="/" v-if="dataForm.subjectID">
        <el-breadcrumb-item>学科管理</el-breadcrumb-item>
        <el-breadcrumb-item>{{ $route.params.name }}</el-breadcrumb-item>
        <el-breadcrumb-item>目录管理</el-breadcrumb-item>
      </el-breadcrumb>
      <el-form
        :inline="true"
        class="demo-form-inline"
        :model="dataForm"
        ref="resetForm"
      >
        <el-form-item label="目录名称" prop="tagName">
          <el-input v-model="dataForm.tagName"></el-input>
        </el-form-item>
        <el-form-item label="状态" style="padding-left: 50px" prop="state">
          <el-select v-model="dataForm.state" placeholder="请选择">
            <el-option
              v-for="item in statusList"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button @click="clearForm('resetForm')">清除</el-button>
          <el-button type="primary" @click="handleFilter">搜索</el-button>
        </el-form-item>
        <el-form-item class="right">
          <el-link
            :underline="false"
            type="primary"
            class="el-icon-back"
            v-if="dataForm.subjectID"
            @click="$router.back()"
            >返回学科</el-link
          >
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
          <el-table-column align="center" label="所属学科">
            <template slot-scope="scope">
              <span>{{ scope.row.subjectName }}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="标签名称">
            <template slot-scope="scope">
              <span>{{ scope.row.tagName }}</span>
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
          <el-table-column align="center" label="状态">
            <template slot-scope="scope">
              <span v-if="scope.row.state == 1">启用</span>
              <span v-else>禁用</span>
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
      <!-- 弹出层组件 -->
      <tags-add
        ref="addTags"
        :formData.sync="dataForm"
        :formBase="formData"
        :tags="dataForm.subjectID"
        v-on:newDataes="getList()"
      ></tags-add>
    </el-card>
  </div>
</template>

<script>
import tagsAdd from '../components/tags-add'
import { status } from '@/api/hmmm/constants.js'
import { list, changeState, remove, detail } from '@/api/hmmm/tags.js'
export default {
  name: 'tags',
  components: {
    tagsAdd
  },
  data () {
    return {
      dataList: [], // 数据列表
      alertText: '', // 数据总条条
      dataForm: {
        tagName: null,
        state: null,
        subjectID: this.$route.params.id,
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
    // 初始化渲染
    async getList () {
      const { data } = await list(this.dataForm)
      this.dataList = data.items
      // console.log(data)
      this.total = data.counts
      this.alertText = `数据共 ${this.total} 条`
      this.listLoading = false
    },
    // 重置
    clearForm () {
      this.$refs.resetForm.resetFields()
      this.getList()
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
    },
    // 修改按钮状态(启用/禁用)
    async handleStatus (row) {
      // 定义一个空变量来存按钮状态
      let status = null
      // 如果等于1就是启用状态
      if (row.state === 0) {
        row.state = 1
        status = '启用'
      } else {
        // 反之
        row.state = 0
        status = '禁用'
      }
      // 把要传的数据存为一个对象
      const data = {
        id: row.id,
        state: row.state
      }
      // 提示消息
      this.$message({
        showClose: true,
        message: '更改为' + status + '状态',
        type: 'success'
      })
      // 调用接口,数据持久化
      const { data: res } = await changeState(data)
      this.state = res
      this.getDataList()
    },
    // 删除
    removeTags (tags) {
      this.$confirm('是否删除此标签 ' + ', 是否继续?', '提示', {
        type: 'warning'
      })
        .then(async () => {
          await remove({ id: tags })
            .then(response => {
              this.$message.success('成功删除了标签' + '!')
              this.dataList.splice(tags, 1)
              this.getList(this.dataForm)
            })
            .catch(response => {
              this.$message.error('删除失败!')
            })
        })
        .catch(() => {
          this.$message.info('已取消操作!')
        })
    },
    query () {
      // 下次打开弹出层时置弹出层
      this.formData = {}
    },
    // 添加目录
    addhandleChange (id) {
      // 下次打开弹出层时置空弹出层数据
      this.query()
      this.$refs.addTags.addtansShow()
    },
    // 修改目录
    edithandleChange (id) {
      // 下次打开弹出层时置空弹出层数据
      this.query()
      this.$refs.addTags.edittansShow()
      // 点击编辑拿到当前数据
      this.hanldeEditForm(id)
    },
    // 获取到下拉框数据
    async hanldeEditForm (objeditId) {
      this.dataForm.id = objeditId
      const { data: res } = await detail({ id: objeditId })
      // 获取详情 给formData
      this.formData = res
    }
  }
}
</script>

<style lang="less" scoped>
.container {
  padding: 0 25px;
  border-radius: 5px;
}
.demo-form-inline {
  padding-top: 25px;
}
.alert {
  margin: 10px 0px;
}
.pagination {
  margin-top: 10px;
}
.pages {
  float: right;
}
.right {
  float: right;
}
.num-box {
  width: 100%;
  height: 20px;
  background-color: #f4f4f5;
  border-radius: 5px;
  margin-bottom: 20px;
  span {
    margin: 2px 0;
    font-size: 10px;
    color: #909399;
  }
}
</style>
