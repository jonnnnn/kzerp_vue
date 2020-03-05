<template>
    <el-dialog :visible.sync="visible" :title="isNew ? $t('views.public.add') : $t('views.public.update')"
               :close-on-click-modal="false" :close-on-press-escape="false" width="55%">
        <el-form :model="dataForm" :rules="rules" ref="dataForm"
                 label-width="120px" :inline="true" labelSuffix="："
                 size="mini" class="tb-matthew">
            <el-form-item prop="id" v-show="false" />
            <el-form-item prop="custVendor" :label="data.form.input.custVendor">
                <el-radio-group v-model="dataForm.custVendor" style="width: 400px">
                    <el-radio label='CUST'>客户</el-radio>
                    <el-radio label='VENDOR'>供应商</el-radio>
                </el-radio-group>
            </el-form-item>
            <el-form-item prop="custno" :label="data.form.input.custno">
                <el-input v-model="dataForm.custno" :placeholder="data.form.input.custno" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="custname" :label="data.form.input.custname">
                <el-input v-model="dataForm.custname" :placeholder="data.form.input.custname" :disabled="dataForm.id"/>
            </el-form-item>

            <el-form-item prop="custidentify" :label="data.form.input.custidentify">
                <el-input v-model="dataForm.custidentify" :placeholder="data.form.input.custidentify" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="contactperson" :label="data.form.input.contactperson">
                <el-input v-model="dataForm.contactperson" :placeholder="data.form.input.contactperson"/>
            </el-form-item>
            <el-form-item prop="contactphone" :label="data.form.input.contactphone">
                <el-input v-model="dataForm.contactphone" :placeholder="data.form.input.contactphone"/>
            </el-form-item>
            <el-form-item prop="custadd" :label="data.form.input.custadd">
                <el-input v-model="dataForm.custadd" :placeholder="data.form.input.custadd"/>
            </el-form-item>
            <el-form-item prop="custtype" :label="data.form.input.custtype">
                <el-input v-model="dataForm.custtype" :placeholder="data.form.input.custtype" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="relation" :label="data.form.input.relation">
                <el-input v-model="dataForm.relation" :placeholder="data.form.input.relation" :disabled="dataForm.id"/>
            </el-form-item>
            <el-form-item prop="isActive" :label="data.form.input.isActive">
                <el-input v-model="dataForm.isActive" :placeholder="data.form.input.isActive"/>
            </el-form-item>
            <el-form-item prop="ownerareatext" :label="data.form.input.ownerareatext">
                <el-input v-model="dataForm.ownerareatext" :placeholder="data.form.input.ownerareatext" />
            </el-form-item>
            <el-form-item prop="purchaser" :label="data.form.input.purchaser">
                <el-input v-model="dataForm.purchaser" :placeholder="data.form.input.purchaser"/>
            </el-form-item>
            <el-form-item prop="businessman" :label="data.form.input.businessman">
                <el-input v-model="dataForm.businessman" :placeholder="data.form.input.businessman"/>
            </el-form-item>
            <el-form-item prop="mainopname" :label="data.form.input.mainopname">
                <el-input v-model="dataForm.mainopname" :placeholder="data.form.input.mainopname"/>
            </el-form-item>
            <el-form-item prop="remark" :label="data.form.input.remark">
                <el-input v-model="dataForm.remark" :placeholder="data.form.input.remark"/>
            </el-form-item>
            <el-form-item prop="companyId" :label="data.form.input.companyId">
                <el-input v-model="dataForm.companyId" :placeholder="data.form.input.companyId" :disabled="dataForm.id"/>
            </el-form-item>
        </el-form>
        <template slot="footer">
            <el-button @click="visible = false">{{ $t('views.public.cancel') }}</el-button>
            <el-button type="primary" @click="dataFormSubmit">{{ $t('views.public.confirm') }}</el-button>
        </template>
    </el-dialog>
</template>

<script>
import data from './data'
import mixinViewModule from '@/mixins/view-module'
export default {
  mixins: [mixinViewModule],
  data () {
    let checkTel = (rule, value, callback) => {
      if (value) {
        let pattern = /0?(13|14|15|17|18|19)[0-9]{9}/
        pattern.test(value) ? callback() : callback(new Error('电话格式不正确'))
      } else {
        callback()
      }
    }

    return {
      mixinViewModuleOptions: {
        getDataListURL: '/base/cust/list',
        updateURL: '/base/cust/save',
        getDataListIsPage: false,
        activatedIsNeed: false
      },
      data: data,
      visible: false,
      dataForm: {
          id: undefined,
          custVendor: 'CUST',
          custno: undefined,
          custname: undefined,
          custidentify: undefined,
          contactperson: undefined,
          contactphone: undefined,
          custadd: undefined,
          custtype: undefined,
          relation: undefined,
          isActive: undefined,
          ownerareatext: undefined,
          purchaser: undefined,
          businessman: undefined,
          mainopname: undefined,
          remark: undefined
      },
      rules: {
        custname: [{
          required: true, message: '名称不可缺少'
        }],
          contactphone: [{
          validator: checkTel, trigger: 'blur'
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
          url: '/base/cust/save',
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
    div.el-radio-group {
        width: 200px
    }
    input.el-input__inner{
        width: 200px;
    }
  }
</style>
