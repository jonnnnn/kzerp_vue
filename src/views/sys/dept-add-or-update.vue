<template>
  <el-dialog class="abow-dialog" 
  :visible.sync="visible" 
  :title="dataForm.deptId==null ? $t('views.public.add') : $t('views.public.update')" 
  :close-on-click-modal="false" 
  :close-on-press-escape="false"
  v-loading.fullscreen.lock="fullscreenLoading"
  >
    <el-form  :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmitHandle()" label-width="120px" size="mini">
      <el-form-item prop="name" :label="$t('views.public.dept.name')" >
        <el-input v-model="dataForm.name" :placeholder="$t('views.public.dept.name')" />
      </el-form-item>
      <el-form-item prop="parentName" :label="$t('views.public.dept.parentId')" class="dept-list" >
        <el-popover v-model="deptListVisible" ref="deptListPopover" placement="bottom-start" trigger="click">
          <el-tree
            :data="deptList"
            :props="{ label: 'name', children: 'children' }"
            node-key="deptId"
            ref="deptListTree"
            :highlight-current="true"
            :expand-on-click-node="false"
            accordion
            @current-change="deptListTreeCurrentChangeHandle">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.parentName" v-popover:deptListPopover :readonly="true" :placeholder="$t('views.public.dept.parentId')"/>
      </el-form-item>
      <el-form-item prop="parentId" :label="$t('views.public.dept.parentId')" v-show="false" >
        <el-input v-model="dataForm.parentId" :placeholder="$t('views.public.dept.parentId')"/>
      </el-form-item>
      <el-form-item prop="orderNum" :label="$t('views.public.dept.orderNum')">
        <el-input v-model="dataForm.orderNum" :placeholder="$t('views.public.dept.orderNum')"/>
      </el-form-item>
      <el-form-item label="类型">
       <template>
      <el-radio-group v-model="dataForm.type">
        <el-radio :label="0">公司</el-radio>
        <el-radio :label="1">部门</el-radio>
      </el-radio-group>
      </template>
      </el-form-item>
      <el-form-item label="设置公司"  v-show="dataForm.currentId===1?true:false">
       <template>
      <el-radio-group v-model="dataForm.isTop">
        <el-radio :label="1">是</el-radio>
        <el-radio :label="0">否</el-radio>
      </el-radio-group>
      </template>
      </el-form-item>
    </el-form>
    <template slot="footer">
      <el-button @click="visible = false">{{ $t('views.public.cancel') }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t('views.public.confirm') }}</el-button>
    </template>
  </el-dialog>
</template>

<script>
import { debounce } from 'lodash'
export default {
  data () {
    return {
      visible: false,
      deptList: [],
      deptListVisible: false,
      fullscreenLoading: false,
      dataForm: {
        deptId: '',
        name: '',
        parentId: '',
        parentName: '',
        orderNum: 0,
        type: 1,
        delFlat:0,
        isTop: 0,
        currentId:''
      }
     
    }
  },
  
  computed: {
      dataRule () {
        var validateDeptName = (rule, value, callback) => {
          if (!this.dataForm.roleId && !/\S/.test(value)) {
            return callback(new Error("部门名称不能为空！"))
          }
          callback()
          }
          var validateParentName = (rule, value, callback) => {
          if (!this.dataForm.roleId && !/\S/.test(value)) {
            return callback(new Error("部门名称不能为空！"))
          }
          callback()
          }
          
      return{
         name: [
          { required: true, message: "部门名称不能为空！", trigger: 'blur' },
          { validator: validateDeptName, trigger: 'blur' }
        ],
        parentName: [
          { required: true, message: "上级部门不能为空！", trigger: 'change' },
          { validator: validateParentName, trigger: 'blur' }
        ]
        
      }
    }
  },
   
  methods: {
    
    init () {
      this.visible = true
      this.dataForm.deptId=null
      this.dataForm.isTop = 0
      this.dataForm.type = 1
      if(this.$refs.dataForm){
        this.$refs.dataForm.resetFields()
      }
      this.$nextTick(() => { 
        Promise.all([
          this.getDeptList()
        ]).then(() => {
         
        })
      })
    },
    
    update(row){
      this.visible = true
      this.$nextTick(() => {
        this.dataForm = Object.assign({}, row)
        this.$refs['dataForm'].clearValidate()
        Promise.all([
          this.getDeptList()
        ]).then(() => {
          if (this.dataForm.deptId) {
             
          }
        })
      })
    },
    // 获取部门列表
    getDeptList () {
      return this.$axios.get('/sys/dept/select').then(res => {
        this.deptList = res
      }).catch(() => {})
    },
    
    // 所属部门树, 选中
    deptListTreeCurrentChangeHandle (data, node) {
      debugger
      this.dataForm.parentId = data.deptId
      this.dataForm.parentName = data.name
      this.deptListVisible = false
    },
    // 表单提交
    dataFormSubmitHandle () {
      this.$refs['dataForm'].validate((valid) => {
        if (!valid) {
          return false
        }
        this.fullscreenLoading = true
        this.$axios.post('/sys/dept/save', {
          ...this.dataForm
        }).then(res => {
          this.fullscreenLoading = false
          this.$message({
            message: this.$t('views.public.success'),
            type: 'success',
            duration: 500,
            onClose: () => {
              this.visible = false
              //this.search()
              this.$emit('refreshDataList')
            }
          })
        }).catch(() => {
          this.fullscreenLoading = false
        })
      })
    }
  }
}
</script>

<style lang="scss">
.mod-sys__user {
  .dept-list {
    .el-input__inner,
    .el-input__suffix {
      cursor: pointer;
    }
  }
  .role-list {
    .el-select {
      width: 100%;
    }
  }
}
</style>
