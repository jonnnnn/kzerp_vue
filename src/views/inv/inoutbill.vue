<template>
  <d2-container >
    <el-collapse slot="header">
      <el-collapse-item>
        <template slot="title">
          查询条件<i class="el-icon-d-arrow-right" />
        </template>
        <dynamic-form
          v-model="dataForm"
          :formprops="formprops"
          ref="dataForm"
          col-span='6,6,*,6'
          :alldescriptors="descriptors">
          <template slot="btnsearch">
            <el-button type="primary" icon="el-icon-search" @click="search" >查询</el-button>
            <el-button icon="el-icon-refresh" @click="reset">重置</el-button>
          </template>
        </dynamic-form>
      </el-collapse-item>
    </el-collapse>

    <vxe-grid
      border
      resizable
      highlight-current-row
      remote-filter
      size="mini"
      ref="pGrid"
      row-id="id"
      :toolbar="toolbar"
      :proxy-config="tableProxy"
      :columns="tableColumn"
      :select-config="{reserve: true}"
      :edit-config="{trigger: 'click', mode: 'row', showStatus: true}"
      @cell-dblclick="cellDblClick"
      @current-change='currentChange'
      :footer-method="footerMethod"
      show-footer
      >
      <template v-slot:buttons>
        <el-button ref="btnStatusPick" type="primary" size="mini" icon="el-icon-user"
        enablestatus='NEW'
        row-dbclick
        form-readonly
        v-if="$hasPermission('inv:inoutbill:save')"
        @click="e => cellDblClick({row: $refs.pGrid.getCurrentRow()}, e)">拣货</el-button>
        <el-button type="info" size="mini" icon="el-icon-printer">打印</el-button>
        <el-button type="info" size="mini" icon="fa fa-file-excel-o" @click="$refs.pGrid.exportCsv()">  导出</el-button>
      </template>
    </vxe-grid>

    <!-- 分页 -->
    <el-pagination
      slot="footer"
      :current-page="page"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="limit"
      :total="total"
      layout="total, sizes, prev, pager, next, jumper"
      @size-change="val => pageSizeChangeHandle(val, 'vxe')"
      @current-change="val => pageCurrentChangeHandle(val, 'vxe')"
    ></el-pagination>

    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="search"></add-or-update>
  </d2-container>
</template>

<script>
import AddOrUpdate from './inoutbill-add-or-update'
import mixinViewModule from '@/mixins/view-module'
import XEUtils from 'xe-utils'

export default {
  name: 'inv-inoutbill',
  mixins: [mixinViewModule],
  data () {
    return {
      mixinViewModuleOptions: {
        getDataListURL: '/inv/inoutbill/list',
        getDataListIsPage: true,
        updateURL: '/inv/inoutbill/save',
        deleteIsBatch: true,
        exportURL: '/inv/inoutbill/export'
      },
      order: 'desc',
      orderField: 'id',
      dataForm: {
        billNum: null,
        transactionType: null,
        sourceOrderNum: null,
        status: 'NEW',
        warehouse: null
      },
      descriptors: {
        billNum: { type: 'string',
          label: '业务单号',
          props: {
            clearable: true
          }
        },
        transactionType: { type: 'cust',
          label: '业务类型',
          name: 'im-selector',
          placeholder: '请选择业务类型',
          props: {
            mapKeyVal: 'transactionType',
            dataType: 'code.tran_type',
            clearable: true
          }
        },
        status: { type: 'cust',
          label: '单据状态',
          placeholder: '请选择状态',
          name: 'im-selector',
          props: {
            mapKeyVal: 'status',
            dataType: 'code.status',
            clearable: true
          }
        },
        btnSearch: {
          type: 'slot',
          name: 'btnsearch'
        },
        separate1: this.$g.separate,
        sourceOrderNum: { type: 'string', label: '单据单号' },
        warehouseId: { type: 'cust',
          label: '仓库',
          ruletype: 'integer',
          name: 'im-selector',
          props: {
            mapKeyVal: 'warehouseCode:warehouseId',
            dataType: 'biz.warehouse',
            clearable: true
          }
        },
        inDate: { type: 'date',
          label: '入库日期',
          props: {
            type: 'daterange',
            rangeSeparator: '至',
            startPlaceholder: '开始日期',
            endPlaceholder: '结束日期',
            valueFormat: 'yyyy-MM-dd'
          }
        }
      },
      tableColumn: [
        { type: 'index', width: 50, align: 'center' },
        {
          title: '业务单号',
          field: 'billNum',
          sortable: true,
          width: 110,
          align: 'center'
        },
        {
          title: '业务类型',
          field: 'transactionTypeMean',
          sortable: true,
          width: 80,
          align: 'center'
        },
        {
          title: '业务单号',
          field: 'sourceOrderNum',
          sortable: true,
          width: 110,
          align: 'center'
        },
        {
          title: '业务单据类型',
          field: 'sourceOrderTypeMean',
          sortable: true,
          width: 100,
          align: 'center'
        },
        {
          title: '经办人',
          field: 'pic',
          sortable: true,
          width: 70,
          align: 'center'
        },
        {
          title: '出入库日期',
          field: 'inDate',
          sortable: true,
          align: 'center'
        },
        {
          title: '状态',
          field: 'statusMean',
          sortable: true,
          width: 70,
          align: 'center'
        },
        {
          title: '出入库仓库',
          field: 'warehouseCode',
          sortable: true,
          width: 100,
          align: 'center'
        },
        {
          title: '修改人',
          field: 'updateBy',
          sortable: true,
          width: 70,
          align: 'center'
        },
        {
          title: '修改日期',
          field: 'updateDate',
          sortable: true,
          align: 'center'
        }
      ],
      toolbar: {
        id: 'inoutbill_toolbar_1',
        refresh: true,
        resizable: {
          storage: true
        },
        setting: {
          storage: true
        }
      }
    }
  },
  components: {
    AddOrUpdate
  },
  methods: {
    initSelData () {
      for (const key in this.descriptors) {
        if (XEUtils.get(this.descriptors[key], 'name') === 'el-date-picker' ||
        XEUtils.get(this.descriptors[key], 'type') === 'date') {
          this.descriptors[key].props.pickerOptions = this.pickerOptions
        }
      }
    },
    handleFormReset () {
      this.$refs.dataForm.resetFields()
    }
  }
}
</script>
<style lang="scss">
</style>
