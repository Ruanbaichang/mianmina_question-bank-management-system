<template>
  <div class="tags-add">
    <!-- 新增 -->
    <el-dialog
      title="新增目录"
      :visible.sync="addTagsShow"
      width="23%"
      style="margin-top: -30px"
    >
      <el-form
        :model="formBase"
        :rules="rules"
        ref="ruleForm"
        label-position="left"
        label-width="120px"
        style="width: 400px"
      >
        <el-form-item
          label="所属学科"
          prop="subjectID"
          v-if="!this.$route.params.id"
        >
          <el-select
            style="width: 280px"
            v-model="formBase.subjectID"
            placeholder="请选择"
          >
            <el-option
              v-for="item in subjectTags"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="目录名称" prop="tagName">
          <el-input
            v-model="formBase.tagName"
            placeholder="请输入目录名称"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addTagsShow = false" class="btnone">取消</el-button>
        <el-button type="primary" @click="addTagsData" class="btntwo">
          确定
        </el-button>
      </span>
    </el-dialog>
    <!-- 修改 -->
    <el-dialog
      title="修改目录"
      :visible.sync="editTagsShow"
      width="23%"
      style="margin-top: -30px"
    >
      <el-form
        :model="formBase"
        :rules="rules"
        ref="ruleForm"
        label-position="left"
        label-width="120px"
        style="width: 400px"
      >
        <el-form-item
          label="所属学科"
          prop="subjectID"
          v-if="!this.$route.params.id"
        >
          <el-select
            style="width: 280px"
            v-model="formBase.subjectID"
            placeholder="请选择"
          >
            <el-option
              v-for="item in subjectTags"
              :key="item.id"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="目录名称" prop="tagName">
          <el-input
            v-model="formBase.tagName"
            placeholder="请输入目录名称"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="editTagsShow = false" class="btnone">取消</el-button>
        <el-button type="primary" @click="editTagsData" class="btntwo"
          >确定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { add, update } from '@/api/hmmm/tags.js'
import { simple } from '@/api/hmmm/subjects.js'
export default {
  name: 'tagsAdd',
  components: {

  },
  props: {
    formBase: {
      type: [Object],
      required: true
    },
    tags: {
      type: [Number]
    }
  },
  data () {
    return {
      addTagsShow: false, // 新增
      editTagsShow: false, // 修改
      subjectTags: {
        subjectID: '',
        tagName: ''
      },
      // 表单验证
      rules: {
        tagName: [
          { required: true, message: '请输入目录名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    console.log(this.tags)
  },
  methods: {
    // 弹出对话框框
    addtansShow () {
      this.addTagsShow = true
      // 弹出的时候渲染下拉框
      this.getSubject()
    },
    // 弹出对话框框
    edittansShow () {
      this.editTagsShow = true
      // 弹出的时候渲染下拉框
      this.getSubject()
    },
    // 获取下拉框标签
    async getSubject () {
      const { data } = await simple()
      this.subjectTags = data
    },
    // 表单提交 新增
    addTagsData () {
      console.log(this.formBase)
      try {
        this.$refs.ruleForm.validate(async valid => {
          if (valid) {
            this.addtansShow()
            await add({
              subjectID: this.tags === undefined ? this.formBase.subjectID : this.tags,
              tagName: this.formBase.tagName
            })
            this.$message.success('添加成功')
            this.$emit('newDataes', this.formBase)
          } else {
            this.$Message.error('*号为必填项!')
          }
          this.addTagsShow = false
        })
      } catch (error) {
        console.error(error)
      }
    },
    // 表单提交 编辑
    editTagsData () {
      console.log(this.formBase)
      try {
        this.$refs.ruleForm.validate(async valid => {
          if (valid) {
            await update(this.formBase)
            this.$message.success('修改成功')
            this.$emit('newDataes', this.formBase)
          } else {
            this.$message.error('*号为必填项!')
          }
          this.editTagsShow = false
        })
      } catch (error) {
        console.error(error)
      }
    }
  }
}
</script>

<style lang="less" scoped>
.btnone {
  margin-left: 250px;
}
.btntwo {
  // float: right;
}
</style>
