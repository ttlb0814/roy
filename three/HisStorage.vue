<template>
  <div>
    <el-row>
      <el-col :span="5" class="bg-fff borderR5">
        <Form onsubmit="return false" :label-width="80" style="width: 100%; display:inline-block" inline class="search-form search-form1">
          <label class="label-sign2" style="margin-left: 10px"></label>
          <Form-item style="width: 88%" label="入库历史">
            <Input
              v-model="formObj.consumeName"
              placeholder="请输入资产名称"
              clearable
              style="width: 100%"
              icon="ios-search-outline"
              @on-click="_formSearch"
              @on-enter="_formSearch"
            />
          </Form-item>
        </Form>
      </el-col>
      <el-col :span="19" class="ilist-style">
        <icon-list
          :msg="ilist"
          @on-result-change="_btnClick"
        >
        </icon-list>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <div class="bg-fff padding-10" style="margin-top: -20px">
          <element-table
            id="hisStorageTable"
            ref="pageTable"
            :page-columns="pageColumns"
            :get-page="getPage"
            show-check-box
            select-data
            :table-height="tableHeight"
            @on-result-change="_tableResultChange"
          >
            <el-table-column
              v-for="item in pageColumns"
              :key="item.key"
              show-overflow-tooltip
              sortable
              :prop="item.key"
              :label="item.title"
              :width="item.width"
              :min-width="200"
              :fixed="item.fixed?item.fixed:undefined"
            >
              <template slot-scope="scope">
                <div v-if="item.date">
                  {{ scope.row[item.key]?$dateformat(scope.row[item.key],'yyyy-mm-dd'):'' }}
                </div>
                <div v-else>
                  {{ scope.row[item.key] }}
                </div>
              </template>
            </el-table-column>
          </element-table>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { inventoryOverview } from '../../../api'

export default {
  components: {},
  data() {
    return {
      id: '',
      name: '',
      formObj: {
        pid: '',
        consumeName: ''
      },
      searchOpen: false,
      getPage: {
        records: []
      },
      pageColumns: [
        { title: '资产名称', key: 'consumeName' },
        { title: '规格型号', key: 'spec' },
        { title: '资产编号', key: 'code' },
        { title: 'CAS编号', key: 'cas' },
        { title: '批次号', key: 'batch' },
        { title: '入库数量', key: 'inStock' },
        { title: '计量单位', key: 'unit' },
        { title: '购买日期', key: 'buyDate', date: true },
        { title: '有效截止日期', key: 'validUntil', date: true },
        { title: '入库人', key: 'inUser' },
        { title: '入库时间', key: 'inTime' },
        { title: '资产品牌', key: 'brand' },
        { title: '单价（元）', key: 'price' },
        { title: '供应商', key: 'supplier' },
        { title: '保存条件', key: 'keep' },
        { title: '保管人', key: 'keeper' },
        { title: '保管人电话', key: 'keeperTel' }
      ],
      ilist: [
        {
          type: 'iconfont icon-daochu',
          color: '#bfbfbf',
          id: '',
          name: '导出'
        }
      ],
      selectIds: [],
      selectData: []
    }
  },
  computed: {
    tableHeight: function() {
      if (this.searchOpen) {
        return this.$tableHeight('search')
      } else {
        return this.$tableHeight('', 240)
      }
    }
  },
  methods: {
    _search() {
      this._page()
    },
    _formSearch() {
      this.$refs.pageTable._pageChange(1)
    },
    async _page() {
      Object.assign(this.formObj, this.$refs.pageTable._searchParams())
      const result = await inventoryOverview.receiveApply(
        this.$serializeForm(this.formObj)
      )
      if (result) {
        this.getPage = result
        this.$refs.pageTable._initTable()
      }
      await console.log(1)
    },
    _tableResultChange(msg, data) {
      switch (msg) {
        case 'page':
          this._page()
          break
        case 'selectData':
          this.selectIds = data.map(({ id }) => id)
          this.selectData = data
          break
        default:
          this._page()
      }
    },
    _btnClick(msg) {
      switch (msg) {
        case 'search':
          this.searchOpen = !this.searchOpen
          break
        case '导出':
          this._export()
          break
      }
    },
    // 导出
    _export() {
      const data = this.selectData
      if (this.getPage.records.length === 0) {
        return this.$message.warning('暂无数据，不可导出')
      } else if (this.getPage.records.length > 0 && data.length === 0) {
        this.$confirm('确定导出全部（最多5000条）数据？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async () => {
          const data = { page: 1, rows: 5000 }
          Object.assign(data, this.$serializeForm(this.formObj))
          const { records } = await inventoryOverview.receiveApply(data)
          this.$exportExcel(
            'hisStorageTable',
            '入库历史导出表',
            this.pageColumns,
            records
          )
        })
      } else {
        this.$confirm('确定导出这' + data.length + '条数据？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$exportExcel(
            'hisStorageTable',
            '入库历史导出表',
            this.pageColumns,
            data
          )
        })
      }
    }
  }
}
</script>

<style scoped>
.Home {
  background-color: #f2f2f2;
}
.search-form1 {
  margin-bottom: 0;
  margin-left: 2px;
  padding-left: 0;
  background-color: #ffffff;
}
.search-form1 >>> .ivu-form-item-label {
  line-height: 17px;
  font-size: 16px;
}
.search-form1 .ivu-form-item {
  margin-top: 10px;
  margin-left: 17px;
}
.search-form1 >>> .ivu-input-wrapper {
  background-color: #f2f2f2;
  border-radius: 5px;
}
.search-form1 >>> .ivu-input {
  height: 26px;
  line-height: 1;
  border: none;
  background-color: #f2f2f2;
}
.label-sign2 {
  position: absolute;
  width: 3px;
  height: 20px;
  background: #ff9900 !important;
  margin-bottom: 0;
  margin-top: 18px;
}
.ilist-style {
  padding-top: 11px;
  padding-right: 55px;
  padding-bottom: 13px;
  border-top-right-radius: 5px;
  background-color: #ffffff;
}
</style>
