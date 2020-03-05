<template>
  <el-dialog :title="isNew ? '销售单新增' : '销售单修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    v-loading.fullscreen.lock="fullscreenLoading"
    class="abow_dialog"
    width="80%"
  >
    <div>
      <el-form
        :model="dataForm"
        labelSuffix="："
        size="mini"
        :rules="dataRule"
        ref="dataForm"
        label-width="120px"
      >
      <el-input v-model="dataForm.orderType" style="display: none"></el-input>

        <el-row >
          <el-col :span="8">
            <el-form-item label="客户" prop="customerId">
              <im-selector
              v-model="dataForm.customerId"
              :mapModel.sync="dataForm"
              mapKeyVal="customerName:customerId"
              dataType="biz.customer"
              placeholder="请选择客户"
              @change="changeCust">
              </im-selector>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="销售日期" prop="orderDate">
              <el-date-picker
                v-model="dataForm.orderDate"
                placeholder="销售日期"
                value-format="yyyy-MM-dd"
              ></el-date-picker>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="要求交货期" prop="planDeliveryDate">
              <el-date-picker
                v-model="dataForm.planDeliveryDate"
                placeholder="要求交货期"
                value-format="yyyy-MM-dd"
              ></el-date-picker>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row >
          <el-col :span="8">
            <el-form-item label="发运方式" prop="shipType">
              <im-selector
              v-model="dataForm.shipType"
              :mapModel.sync="dataForm"
              mapKeyVal="shipType"
              dataType="dict.SHIP_TYPE"
              @change="changeCust">
              </im-selector>
            </el-form-item>
          </el-col>
          <!--<el-col :span="8">
            <el-form-item label="仓库" prop="autoPeaking">
              <el-checkbox v-model="dataForm.autoPeaking">是否自动拣货</el-checkbox>
            </el-form-item>
          </el-col>-->
          <el-col :span="8">
            <el-form-item label="订单金额" prop="orderAmount">
              <el-input v-model="dataForm.orderAmount" disabled placeholder="订单金额" clearable></el-input>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row >
          <el-col :span="8">
            <el-form-item label="收货人" prop="receiveName">
              <el-input v-model="dataForm.receiveName" placeholder="收货人" clearable></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="收货人电话" prop="receivePhone">
              <el-input v-model="dataForm.receivePhone" placeholder="收货人电话" clearable></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="收货地址" prop="receiveAddress">
              <el-input v-model="dataForm.receiveAddress" placeholder="收货地址" clearable></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row >
          <el-col >
        <el-form-item label="备注" prop="remark">
          <el-input style="width:100%" v-model="dataForm.remark" placeholder="备注"></el-input>
        </el-form-item>
          </el-col>
        </el-row>
      </el-form>

    </div>
    <vxe-grid
      border
      resizable
      size="mini"
      highlight-current-row
      class="vxe-table-element"
      remote-filter
      ref="sGrid"
      :toolbar="toolbar"
      :proxy-config="tableProxy"
      :edit-rules="validRules"
      :columns="itableColumn"
      :select-config="{reserve: true}"
      :mouse-config="{selected: true}"
      :keyboard-config="{isArrow: true, isDel: true, isTab: true, isEdit: true}"
      :edit-config="{trigger: 'dblclick', mode: 'cell'}"
      :footer-cell-class-name="footerCellClassName"
      :footer-method="footerMethod"
      show-footer
    >
    <template v-slot:buttons>
        <el-button size="mini" icon="el-icon-circle-plus" @click="$refs.sGrid.insert({})">新增</el-button>
        <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeSelecteds($refs.sGrid)">删除</el-button>
    </template>
    </vxe-grid>

    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" :disabled="btnDisable" @click="dataFormSubmit">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import mixinViewModule from '@/mixins/view-module'
import XEUtils from 'xe-utils'
export default {
  mixins: [mixinViewModule],
  data () {
    return {
      restaurants: [],
      state1: '',
      state2: '',

      mixinViewModuleOptions: {
        getDataListURL: '/so/salesorderline/list',
        getDataListIsPage: false,
        updateURL: '/so/salesorder/update',
        deleteIsBatch: true,
        prodURL: '/base/product/search'
      },

      visible: false,
      btnDisable: false,
      dataForm: {
        id: undefined,
        orderNum: "",
        orderType: "SO",
        customerId: "",
        orderDate: new Date(),
        planDeliveryDate: new Date(),
        orderAmount: "0",
        shipType: "",
        remark: "",
        autoPeaking: true,
        deletedFlag: "N",
        receiveName:"",
        receivePhone:"",
        receiveAddress:""
      },
      dataRule: {
        customerId: [
          { required: true, message: "客户不能为空", trigger: "blur" }
        ],
        orderDate: [
          { required: true, message: "销售日期不能为空", trigger: "blur" }
        ]
      },
      tableProxy: {
        autoLoad: false
      },
      validRules: {
        name: [
          { required: true, message: '商品必填' }
        ],
        prodno: [
          { required: true, message: '商品必填' }
        ],
        orderQty: [
          { required: true, message: '销售数量必填'},
          { type:"number",message: '请输入数字'}
        ],
        price: [
          { required: true, message: '销售价格必填' },
          { type:"number",message: '请输入数字'}
        ]
      },
      itableColumn: [
        { type: "selection", width: 50, align: "center" },
        { type: "index", width: 50, align: "center" },
        {
          title: '商品编码',
          field: 'prodno',
          width: '110px',
          align: 'center'
        },
        {
          title: '商品名称',
          field: 'name',
          width: '150px',
          align: 'center',
          editRender: {
            name: 'ElAutocomplete',
            props: { placeholder:"请输入内容",fetchSuggestions: this.prodSeach, triggerOnFocus: false ,popperClass:'prod-popper' },
            events: { select: this.handleProcSelect },
            autoselect: true
          }
        },
        {
          title: '单位',
          field: 'unit',
          width: '40px',
          align: 'center'
        },
        {
          title: '规格',
          field: 'specialParam',
          width: '100',
          align: 'center'
        },
        {
          title: '生产厂家',
          field: 'manufacture',
          width: '110px',
          align: 'left'
        },{
          title: '批准文号',
          field: 'approvalno',
          width: '110px',
          align: 'left'
        },{
          title: '业务类型',
          field: 'busitypetext',
          width: '110px',
          align: 'center'
        },{
          title: '灭菌批号',
          field: 'sterilization',
          width: '110px',
          align: 'center'
        },{
          title: '数量',
          field: 'orderQty',
          align: 'center',
          width: '60px',
          editRender: { name: 'input' ,autoselect: true}
        },
        {
          title: '销售价',
          field: 'price',
          sortable: true,
          align: 'center',
          width: '100px',
          editRender: { name: 'input',autoselect: true},
          footerRender: function (column, data) {
            return '汇总'
          }
        },
        {
          title: '总金额',
          field: 'amount',
          align: 'left',
          width: '100px',
          formatter: ['toFixedString', 2],
          needReturnAmount : true,
          editPost: function (column, row) {
            var qty = row.orderQty
            var price = row.price
            if (!Number.isNaN(qty) && !Number.isNaN(price)) {
              return Number(qty) * Number(price).toFixed(2)
            }
          },
          footerRender: function (column, data) {
            return XEUtils.sum(data, column.property)
          }
        },{
          title: '备注',
          field: 'note',
          align: 'center',
          width: '130px',
          editRender: { name: 'input' ,autoselect: true}
        }

      ],
      toolbar: {
        id: 'full_edit_1',
        resizable: {
          storage: true
        },
        setting: {
          storage: true
        }
      }
    };
  },
  methods: {

    prodSeach (queryString, cb) {

      if (queryString) {
        this.$axios
          .post(this.mixinViewModuleOptions.prodURL, { name: queryString ,customerId:this.dataForm.customerId,lastPrice:1})
          .then(res => {
            for (var i = 0; i < res.length; i++) {
              res[i].value = res[i].allString;
            }
            clearTimeout(this.timeout)
            this.timeout = setTimeout(() => {
              cb(res);
            }, 100 * Math.random())
          })
      }

    },
    handleProcSelect(t, item) {
      var row = t.row;
      if (item) {
        Object.assign(row, item);
        row.productId = item.id;
        row.productName = item.name;
        row.stock = item.stock;
        row.bPrice = item.salePrice;
        row.productCode = item.code;
        row.price = item.salePrice;
        this.$refs.sGrid.updateFooter()
      } else {
      }
    },
    changeCust (e) {
      console.log('------', e, this.dataForm)
    },
    setAmount(value){
      this.dataForm.orderAmount = value;
    },

    mounted() {

    }

  }

}
</script>
