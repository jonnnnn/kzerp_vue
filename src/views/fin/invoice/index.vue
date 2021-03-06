<template>
    <d2-container class="mod-sys__user">
        <el-collapse slot="header" v-model="activeName">
            <el-collapse-item name="1">
                <template slot="title">
                    查询条件<i class="el-icon-d-arrow-right"/>
                </template>
                <el-form :inline="true" size="mini" @keyup.enter.native="search" :model="dataForm"
                         ref="dataForm">
                    <el-form-item prop="code">
                        <el-input
                                v-model="dataForm.code"
                                :placeholder="data.form.invoice.code"
                                clearable
                        />
                    </el-form-item>
                    <el-form-item prop="customerName">
                        <el-input
                                v-model="dataForm.customerName"
                                :placeholder="data.form.invoice.customerName"
                                clearable
                        />
                    </el-form-item>
                    <el-form-item prop="from">
                        <el-date-picker
                                v-model="dataForm.from"
                                type="date"
                                valueFormat="yyyy-MM-dd"
                                placeholder="票据日期起">
                        </el-date-picker>
                    </el-form-item>
                    <el-form-item prop="to">
                        <el-date-picker
                                v-model="dataForm.to"
                                type="date"
                                valueFormat="yyyy-MM-dd"
                                placeholder="止">
                        </el-date-picker>
                    </el-form-item>
                    <el-form-item>
                        <el-button @click="search" icon="el-icon-search" type="primary">{{
                            $t('views.public.query') }}
                        </el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button @click="handleFormReset">
                            <d2-icon name="refresh"/>
                            重置
                        </el-button>
                    </el-form-item>
                </el-form>
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
                :columns="columns"
                :select-config="{reserve: true}"
                :edit-config="{trigger: 'click', mode: 'row', showStatus: true}"
                @cell-dblclick="cellDblClick"
                :tree-config="{children: 'children'}"
        >
            <template v-slot:buttons>
                <el-button
                        ref="btnAdd"
                        size="mini"
                        icon="el-icon-circle-plus"
                        @click="addHandle"
                >新增
                </el-button>
                <el-button
                        ref="btnEdit"
                        type="primary"
                        size="mini"
                        icon="el-icon-edit"
                        @click="updateHandle($refs.pGrid)"
                >修改
                </el-button>
                <el-button
                        ref="btnDelete"
                        type="danger"
                        size="mini"
                        icon="el-icon-delete"
                        @click="deleteHandleSetter($refs.pGrid)"
                >删除
                </el-button>
            </template>
        </vxe-grid>
        <el-pagination
        slot="footer"
        :current-page="page"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="limit"
        :total="total"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="val => pageSizeChangeHandle(val, 'vxe')"
        @current-change="val => pageCurrentChangeHandle(val, 'vxe')"/>
        <!-- 弹窗, 新增 / 修改 -->
        <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="search"/>
    </d2-container>
</template>

<script>
import mixinViewModule from '@/mixins/view-module'
import AddOrUpdate from './add-or-update'
import data from '../data'

export default {
  name: 'fin-invoice',
  mixins: [mixinViewModule],
  data () {
    return {
      activeName: '1',
      data: data,
      mixinViewModuleOptions: {
        getDataListURL: '/fin/invoice/list',
        getDataListIsPage: true,
        deleteURL: '/fin/invoice/delete',
        deleteIsBatch: true,
        deleteIsBatchKey: 'id'
      },
      dataForm: {
        code: undefined,
        customerName: undefined,
        from: undefined,
        to: undefined
      },
      toolbar: {
        id: 'full_edit_1',
        refresh: true,
        resizable: {
          storage: true
        },
        setting: {
          storage: true
        }
      },
      columns: [
        { type: 'index', width: 30 },
        {
          title: '编号',
          field: 'code',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '票据名称',
          field: 'name',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '往来单位',
          field: 'customerName',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '票据日期',
          field: 'ivDate',
          sortable: true,
          align: 'center',
          width: '110px',
          formatter: ['toDateString', 'yyyy-MM-dd']
        }, {
          title: '票据金额',
          field: 'amount',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '摘要',
          field: 'summary',
          sortable: true,
          align: 'center',
          width: '110px'
        },

        {
          title: '票据类型',
          field: 'ivName',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '公司名称',
          field: 'cpName',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '税号',
          field: 'taxNum',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '单位地址',
          field: 'apAddress',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '电话号码',
          field: 'cpPhone',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '开户银行',
          field: 'cpBank',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '银行账户',
          field: 'cpBankNum',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '状态',
          field: 'status',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '备注',
          field: 'remark',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '更新人',
          field: 'updateBy',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '更新日期',
          field: 'updateDate',
          sortable: true,
          align: 'center',
          width: '110px',
          formatter: ['toDateString', 'yyyy-MM-dd']
        }
      ]
    }
  },
  components: {
    AddOrUpdate
  },
  methods: {
    handleFormReset () {
      this.$refs['dataForm'].resetFields()
    }
  }
}
</script>

<style>

</style>
