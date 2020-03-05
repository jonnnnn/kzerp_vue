<template>
    <d2-container class="mod-sys__user">
        <el-collapse slot="header">
            <el-collapse-item name="1">
                <template slot="title">
                    查询条件<i class="el-icon-d-arrow-right"/>
                </template>
                <el-form :inline="true" size="mini" :model="dataForm" @keyup.enter.native="search" ref="dataForm">

                    <el-form-item prop="prodno">
                        <el-input
                                v-model="dataForm.prodno"
                                :placeholder="data.form.input.prodno"
                                clearable
                        />
                    </el-form-item>
                    <el-form-item prop="name">
                        <el-input
                                v-model="dataForm.name"
                                :placeholder="data.form.input.name"
                                clearable
                        />
                    </el-form-item>
                    <el-form-item prop="vehicleId">
                        <el-input
                                v-model="dataForm.vehicleId"
                                :placeholder="data.form.input.vehicleId"
                                clearable
                        />
                    </el-form-item>
                    <el-form-item prop="approvalno">
                        <el-input
                                v-model="dataForm.approvalno"
                                :placeholder="data.form.input.approvalno"
                                clearable
                        />
                    </el-form-item>
                    <el-form-item prop="brandId">
                        <el-input
                                v-model="dataForm.brandId"
                                :placeholder="data.form.input.brandId"
                                clearable
                        />
                    </el-form-item>
                    <el-form-item prop="madeinId">
                        <el-input
                                v-model="dataForm.madeinId"
                                :placeholder="data.form.input.madeinId"
                                clearable
                        />
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
        <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="search"/>
    </d2-container>
</template>

<script>
import mixinViewModule from '@/mixins/view-module'
import AddOrUpdate from './add-or-update'
import data from './data'

export default {
  name: 'product',
  mixins: [mixinViewModule],
  data () {
    return {
      activeName: '1',
      data: data,
      mixinViewModuleOptions: {
        getDataListURL: '/base/product/list',
        getDataListIsPage: true,
        deleteURL: '/base/product/delete',
        deleteIsBatch: true
      },
      dataForm: {
        prodno: undefined,
        name: undefined,
        manufacture:undefined,
        approvalno: undefined
      },
      rowHandler: {
        width: '160px',
        custom: [
          {
            text: this.$t('views.public.update'),
            type: 'primary',
            size: 'mini',
            emit: 'user-update',
            show: (index, row) => {
              return this.$hasPermission('sys:user:update')
            }
          },
          {
            text: this.$t('views.public.delete'),
            type: 'danger',
            size: 'mini',
            emit: 'user-delete',
            show: (index, row) => {
              return this.$hasPermission('sys:user:delete')
            }
          }
        ]
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
          title: '商品编码',
          field: 'prodno',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '商品名称',
          field: 'name',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '生产厂家',
          field: 'manufacture',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '批准文号',
          field: 'approvalno',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '大包装数量',
          field: 'bigpackagequantity',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '中包装数量',
          field: 'midpackagequantity',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '业务类型',
          field: 'busitypetext',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '采购员',
          field: 'purchaser',
          sortable: true,
          align: 'center',
          width: '110px'
        }, {
          title: '规格属性',
          field: 'specialParam',
          sortable: true,
          align: 'center',
          width: '120px'
        },
        {
          title: '包装单位',
          field: 'unit',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '灭菌批号',
          field: 'sterilization',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '默认供应商',
          field: 'defaultVendorId',
          sortable: true,
          align: 'center',
          width: '140px'
        }, {
          title: '商品状态',
          field: 'status',
          sortable: true,
          align: 'center',
          width: '120px'
        }, {
          title: '助记码',
          field: 'pinyinCode',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '备注',
          field: 'remark',
          sortable: true,
          align: 'center',
          width: '200px'
        }, {
          title: '修改人',
          field: 'updateBy',
          sortable: true,
          align: 'center',
          width: '110px'
        },
        {
          title: '修改日期',
          field: 'updateDate',
          sortable: true,
          align: 'center',
          width: '120px',
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
