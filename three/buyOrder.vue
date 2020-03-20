<template>
  <div>
    <Modal
      v-model="showEditModal"
      :mask-closable="false"
      :title="modalTitle"
      width="1000px"
    >
      <p slot="header">
        {{ modalTitle }}
      </p>
      <div>
        <Row>
          <!-- 表格 -->
          <Col span="24">
          <element-table
            ref="pageTable"
            :page-columns="pageColumns"
            :table-height="tableHeight"
            :get-page="getPage"
            :icon-msg="iconMsg"
            opt-col-width="60"
            select-data
            show-check-box
            @on-result-change="_tableResultChange"
          >
            <el-table-column
              v-for="item in pageColumns"
              :key="item.key"
              show-overflow-tooltip
              sortable
              :prop="item.key"
              :label="item.title"
              :min-width="item.width"
              :fixed="item.fixed?item.fixed:undefined"
            >
              <template slot-scope="scope">
                <div v-if="item.date">
                  {{ scope.row[item.key]?$dateformat(scope.row[item.key],'yyyy-mm-dd '):'' }}
                </div>
                <div v-else-if="item.key==='brand'" @click.stop="_handleRow(scope)">
                  <Input
                    v-model="scope.row.brand"
                    placeholder=""
                    @on-focus="_handleRow(scope)"
                    @on-blur="_save(item.key)"
                  />
                </div>
                <div v-else-if="item.key==='budget'" @click.stop="_handleRow(scope)">
                  <InputNumber
                    v-model="scope.row.budget"
                    placeholder=""
                    :min="0"
                    @on-focus="_handleRow(scope)"
                    @on-blur="_save(item.key)"
                  />
                </div>
                <div v-else-if="item.key==='quantity'" @click.stop="_handleRow(scope)">
                  <InputNumber
                    v-model="scope.row.quantity"
                    placeholder=""
                    :min="0"
                    :formatter="formatter"
                    :parser="parser"
                    @on-focus="_handleRow(scope)"
                    @on-blur="_save(item.key)"
                  />
                </div>
                <div v-else-if="item.key==='deadline'" @click.stop="_handleRow(scope)">
                  <InputNumber
                    v-model="scope.row.deadline"
                    placeholder=""
                    :min="0"
                    @on-focus="_handleRow(scope)"
                    @on-blur="_save(item.key)"
                  />
                </div>
                <div v-else-if="item.key==='detail'" @click.stop="_handleRow(scope)">
                  <Input
                    v-model="scope.row.detail"
                    placeholder=""
                    @on-focus="_handleRow(scope)"
                    @on-blur="_save(item.key)"
                  />
                </div>
                <div v-else>
                  {{ scope.row[item.key] }}
                </div>
              </template>
            </el-table-column>
          </element-table>
          </Col>
        </Row>
      </div>
      <div slot="footer" class="btn-width">
        <modal-footer ref="footerModal" :footer="footerList" @on-result-change="_footerResult" />
      </div>
    </Modal>
  </div>
</template>
<script>
import modalFooter from '../../../components/base/modalFooter'
import { inventoryOverview } from '../../../api'
import global from '../../../api/config'

export default {
  components: {
    modalFooter
  },
  data() {
    return {
      uid: '',
      orderId: '',
      purchaseOrder: {},
      orderStatus: 0,
      queryData: {},
      formObj: {},
      getPage: {
        records: []
      },
      quantity: '',
      budget: '',
      remark: '',
      modalTitle: '我的申购单',
      showEditModal: false,
      min: 1,
      currentRow: {},
      status: -1,
      list: [],
      selectIds: [],
      selectData: [],
      DATA: {},
      pageColumns: [
        { title: '资产名称', key: 'name', width: 95 },
        { title: '规格型号', key: 'spec', width: 100 },
        { title: '申购品牌', key: 'brand', width: 100, brand: true },
        { title: '库存数量', key: 'stock', width: 100 },
        { title: '申购数量', key: 'quantity', width: 100, quantity: true },
        { title: '申购预算（元）', key: 'budget', width: 130, budget: true },
        {
          title: '到货期限（天）',
          key: 'deadline',
          width: 130
        },
        { title: '申购说明', key: 'detail', width: 155, remark: true }
      ],
      isShow: false,
      isSubmit: false,
      iconMsg: [{ type: 'iconfont icon-shanchu1', id: '', name: '删除' }],
      footerList: [
        { id: '', type: '', name: '取消' },
        { id: '', type: 'warning', color: '#ff9900', name: '提交审批' }
      ],
      footerList1: [{ id: '', type: '', name: '取消' }]
    }
  },
  computed: {
    tableHeight: function() {
      return this.$tableHeight('', 450)
    }
  },
  mounted() {
    this._getUserInfo()
  },
  methods: {
    _getUserInfo() {
      this.uid = global.getUserInfo().id
    },
    _handleRow(scope) {
      this.currentRow = scope.row
    },
    // handleInput(val) {
    //   console.log(val)
    //   const num = toString(val)
    //   // 通过正则过滤小数点后两位
    //   val = num.match(/^\d*(\.?\d{0,1})/g)[0] || null
    // },
    _tableResultChange(msg, data) {
      switch (msg) {
        case 'page':
          this._page()
          break
        case 'selectData':
          this.selectIds = data.map(({ id }) => id)
          this.selectData = data
          break
        case 'iconClick':
          this._iconClick(data.name, data.rowData)
          break
      }
    },
    _open() {
      this.showEditModal = true
      // 待审批显示 " 提交审批 " 按钮
      // 刷新界面
      this._page()
    },
    async _page() {
      const data = { page: 1, rows: 20, status: 'TEMP_SAVE', uid: this.uid }
      // Object.assign(data, this.status)
      const result = await inventoryOverview.applyOrder(data)
      if (result) {
        this.getPage = result
        this.$emit('on-result-change', result.total)
        this._addKey(result.records)
      }
    },
    _iconClick(msg, data) {
      switch (msg) {
        case '删除':
          this._delete([data.id])
          break
      }
    },
    _delete(ids) {
      console.log(ids)
      this.$confirm('确定要删除？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const result = await inventoryOverview.buyDelete(ids)
        if (result) {
          this.$message.success('删除成功！')
          this._open()
        }
      })
    },
    // 格式化数字，输入整数时不填充 '.00'
    formatter(value) {
      const num = value.toString()
      if (num.indexOf('.') !== -1) {
        const arr = num.split('.')
        console.log(value, arr)
        if (arr[1] === '00') {
          return arr[0]
        }
      }
      return value
    },
    parser(value) {
      return value
    },
    _footerResult(msg) {
      switch (msg) {
        case '取消':
          this.showEditModal = false
          break
        case '提交审批':
          this.$refs.footerModal._hideLoading()
          this._submitApprove()
          break
      }
    },
    _addKey(data) {
      data.map(item => {
        if (!item.budget) {
          item.budget = 1
        }
        if (!item.deadline) {
          item.deadline = 7
        }
      })
    },
    _submitApprove() {
      const data = this.selectData
      if (data.length === 0)
        return this.$message.warning('请至少选择一条数据！')
      for (let i = 0; i < data.length; i++) {
        if (!data[i].quantity || data[i].quantity === '') {
          return this.$message.warning('申购数量不可为空，请完善！')
        } else if (!data[i].budget || data[i].budget === '') {
          return this.$message.warning('申购预算不可为空，请完善！')
        } else if (!data[i].deadline || data[i].deadline === '') {
          return this.$message.warning('到货期限不可为空，请完善！')
        } else if (!data[i].detail || data[i].detail === '') {
          return this.$message.warning('申购说明不可为空，请完善！')
        }
      }
      this.$confirm('确定提交审批？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const result = await inventoryOverview.buyOrderSubmit(data)
        if (result) {
          this.$message.success('提交成功！')
          this._page()
          this.showEditModal = false
        }
      })
    },
    _cancel() {
      this.showEditModal = false
    },
    async _save(key) {
      // 保存
      const obj = {
        id: this.currentRow.id,
        obj: { [key]: this.currentRow[key] }
      }
      await console.log(obj)
      // await inventoryOverview.applyOrderEdit(obj)
      // if (result) {
      //   this._page()
      // }
    }
  }
}
</script>
