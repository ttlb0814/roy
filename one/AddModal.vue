<template>
  <div>
    <Modal
      v-model="showEditModal"
      :mask-closable="false"
      width="33%"
      :title="modalTitle"
    >
      <div>
        <Form id="inventory-edit-form" ref="formObj" :model="formObj" :rules="ruleValidate" :label-width="80">
          <Form-item label="类别名称" prop="name" style="margin-bottom: 20px">
            <el-input
              v-model="formObj.name"
              name="name"
              :maxlength="100"
              placeholder="请输入类别名称"
            ></el-input>
          </Form-item>
          <Form-item label="上级类别" prop="pname">
            <el-input
              v-model="formObj.pname"
              name="pname"
              placeholder="请选择上级类别"
              icon="plus-circled"
              readonly
              @click.native="_selectZtree"
            ></el-input>
            <input v-model="formObj.pid" type="hidden" name="pid">
          </Form-item>
          <Form-item label="描述" prop="detail">
            <el-input
              v-model="formObj.detail"
              name="name"
              placeholder="请输入描述"
              :maxlength="100"
              type="textarea"
              :rows="5"
            ></el-input>
          </Form-item>
        </Form>
      </div>
      <div slot="footer">
        <modalFooter ref="footerModal" :footer="footerList" @on-result-change="_footerResult"></modalFooter>
      </div>
    </Modal>
    <!--上级类别弹出树-->
    <AddTree ref="ztreeModal" @on-result-change="_ztree"></AddTree>
  </div>
</template>
<script>
/**
 * 添加编辑
 */
import modalFooter from '../../../../components/base/modalFooter'
import { inventoryOverview } from '../../../../api'
import AddTree from './AddTree'

export default {
  components: {
    AddTree,
    modalFooter
  },
  data() {
    return {
      id: '',
      modalTitle: '添加编辑分类',
      formObj: {
        name: '',
        pid: '',
        pname: '',
        detail: ''
      },
      ruleValidate: {
        name: [{ required: true, message: '类别名称不能为空', trigger: 'blur' }]
      },
      showEditModal: false,
      pname: '',
      footerList: [
        { id: '', type: '', name: '取消' },
        { id: '', type: 'warning', color: '#ff9900', name: '保存' }
      ]
    }
  },
  methods: {
    _resultChange(result, msg) {
      if (result) {
        this.showEditModal = false
        this.$message.success(msg)
        this.$emit('on-result-change')
        // this.$bus.$emit('class', 'class')
      }
    },
    _footerResult(msg) {
      switch (msg) {
        case '取消':
          this._cancel()
          break
        case '保存':
          this._ok()
          break
      }
    },
    _ok() {
      this.$refs.formObj.validate(async valid => {
        if (valid) {
          const data = this.$serializeForm(this.formObj)
          if (this.$string(this.id).isEmpty()) {
            // 添加
            const result = await inventoryOverview.classfiyAdd(data)
            if (result) {
              this._resultChange(result, '添加成功!')
              this.id = ''
            }
          } else {
            // 编辑
            const result = await inventoryOverview.classfiyEdit({
              id: this.id,
              obj: data
            })
            if (result) {
              this._resultChange(result, '编辑成功!')
              this.id = ''
            }
          }
        } else {
          this.$message.error('表单验证失败!')
        }
      })
    },
    _cancel(name) {
      this.showEditModal = false
      this.id = ''
    },
    _open(formObj) {
      this.showEditModal = true
      console.log(formObj)
      this.$nextTick(() => {
        this.$refs.formObj.resetFields()
        if (this.$string(formObj).isEmpty()) {
          this.formObj.pname = ''
          this.formObj.pid = ''
          this.modalTitle = '添加分类'
        } else {
          this.id = formObj.id
          for (const key in this.formObj) {
            this.formObj[key] = formObj[key]
          }
          this.modalTitle = '编辑分类'
        }
      })
    },
    _selectZtree() {
      if (this.$string(this.id).isEmpty()) {
        this.$refs.ztreeModal._openZtree() // 打开上ztreeModel
      } else {
        this.$refs.ztreeModal._openZtree(this.formObj.pid) // 打开上ztreeModel
      }
    },
    _ztree(result) {
      if (result) {
        this.formObj.pname = ''
        this.formObj.pid = result.id
        this.formObj.pname = result.name
      } else {
        this.formObj.pname = ''
        this.formObj.pid = ''
        this.formObj.pname = ''
      }
    }
  }
}
</script>
