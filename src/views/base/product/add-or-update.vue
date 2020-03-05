<template>
    <el-dialog :visible.sync="visible" :title="isNew ? $t('views.public.add') : $t('views.public.update')"
               :close-on-click-modal="false" :close-on-press-escape="false" width="750px">
        <el-form :model="dataForm" :rules="rules" ref="dataForm" label-width="120px" :inline="true" labelSuffix="："
                 size="mini" class="tb-matthew">
            <el-form-item prop="id" v-show="false" />
            <el-form-item prop="prodno" :label="data.form.input.prodno">
                <el-input v-model="dataForm.prodno" :placeholder="data.form.input.prodno" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="name" :label="data.form.input.name">
                <el-input v-model="dataForm.name" :placeholder="data.form.input.name" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="manufacture" :label="data.form.input.manufacture">
                <el-input v-model="dataForm.manufacture" :placeholder="data.form.input.manufacture" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="approvalno" :label="data.form.input.approvalno">
                <el-input v-model="dataForm.approvalno" :placeholder="data.form.input.approvalno" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="busitypetext" :label="data.form.input.busitypetext">
                <el-input v-model="dataForm.busitypetext" :placeholder="data.form.input.busitypetext" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="midpackagequantity" :label="data.form.input.midpackagequantity">
                <el-input v-model="dataForm.midpackagequantity" :placeholder="data.form.input.midpackagequantity" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="bigpackagequantity" :label="data.form.input.bigpackagequantity">
                <el-input v-model="dataForm.bigpackagequantity" :placeholder="data.form.input.bigpackagequantity" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="purchaser" :label="data.form.input.purchaser">
                <el-input v-model="dataForm.purchaser" :placeholder="data.form.input.purchaser"/>
            </el-form-item>
            <el-form-item prop="specialParam" :label="data.form.input.specialParam">
                <el-input v-model="dataForm.specialParam" :placeholder="data.form.input.specialParam" :disabled="dataForm.id"/>
            </el-form-item>

            <el-form-item prop="unit" :label="data.form.input.unit">
                <el-input v-model="dataForm.unit" :placeholder="data.form.input.unit" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="sterilization" :label="data.form.input.sterilization">
                <el-input v-model="dataForm.sterilization" :placeholder="data.form.input.sterilization" :disabled="dataForm.id"/>
            </el-form-item>

            <!--<el-form-item prop="defaultVendorId" :label="data.form.input.defaultVendorId">
                <im-selector
                        placeholder="请选择供应商"
                        v-model="dataForm.defaultVendorId"
                        :mapModel.sync="dataForm"
                        mapKeyVal="dName:defaultVendorId"
                        dataType="biz.customer"
                        style="width: 200px">
                </im-selector>
            </el-form-item>-->
            <el-form-item prop="status" :label="data.form.input.status">
                <el-input v-model="dataForm.status" :placeholder="data.form.input.status"/>
            </el-form-item>
            <el-form-item prop="remark" :label="data.form.input.remark">
                <el-input v-model="dataForm.remark" :placeholder="data.form.input.remark"/>
            </el-form-item>
        </el-form>
        <template slot="footer">
            <el-button @click="visible = false">{{ $t('views.public.cancel') }}</el-button>
            <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t('views.public.confirm') }}</el-button>
        </template>
    </el-dialog>
</template>

<script>
import data from './data'
import mixinViewModule from '@/mixins/view-module'

export default {
  mixins: [mixinViewModule],
  data () {
    let codeValidator = (rule, value, callback) => {
      if (!value) {
        callback(new Error('内容不能为空'))
      } else {
        let form = {}
        if (this.dataForm.name) {
          form.name = this.dataForm.name
        }
        if (this.dataForm.prodno) {
          form.prodno = this.dataForm.prodno
        }
        if (this.dataForm.id) {
          form.id = this.dataForm.id
        }
        this.$axios({
          url: '/base/product/checkCode',
          method: 'post',
          data: form
        }).then(res => {
          if (res) {
            callback(new Error(res))
          } else {
            callback()
          }
        })
      }
    }
    let nameValidator = (rule, value, callback) => {
      if (!value) {
        callback(new Error('内容不能为空'))
      } else {
        let form = {}
        form.name = value
        if (this.dataForm.id) {
          form.id = this.dataForm.id
        }
        this.$axios({
          url: '/base/product/checkCode',
          method: 'post',
          data: form
        }).then(res => {
          if (res) {
            callback(new Error(res))
          } else {
            callback()
          }
        })
      }
    }
    return {
      mixinViewModuleOptions: {
        getDataListURL: '/base/product/list',
        updateURL: '/base/product/save',
        getDataListIsPage: false,
        activatedIsNeed: false
      },
      data: data,
      visible: false,
      dataForm: {
        id: undefined,
        prodno: undefined,
        name: undefined,
        manufacture: undefined,
        approvalno: undefined,
        bigpackagequantity: undefined,
        midpackagequantity: undefined,
        busitypetext: undefined,
        purchaser: undefined,
        sterilization: undefined,
        specialParam: undefined,
        unit: undefined,
        defaultVendorId: undefined,
        status: undefined,
        pinyinCode: undefined,
        remark: undefined
      },
      rules: {
        cateCode: [{
          validator: nameValidator, trigger: 'blur'
        }],
        code: [{
          validator: codeValidator, trigger: 'blur'
        }],
        barCode: [{
          validator: codeValidator, trigger: 'blur'
        }]
      }
    }
  },
  methods: {
    // 表单提交
    dataFormSubmitHandle () {
      let th = this
      this.$refs['dataForm'].validate(valid => {
        if (!valid) {
          return false
        }
        th.$axios({
          url: '/base/product/save',
          method: 'post',
          data: th.dataForm
        }).then(res => {
          this.$message({
            message: this.$t('views.public.success'),
            type: 'success',
            duration: 500,
            onClose: () => {
              this.visible = false
              this.$emit('refreshDataList')
            }
          })
        }).catch(() => {
        })
      })
    }
  }
}
</script>

<style lang="scss">
    .tb-matthew{
        .el-form-item{
            input.el-input__inner{
                width: 200px;
            }
            div.el-radio-group{
                width: 200px;
            }
        }
    }
</style>
