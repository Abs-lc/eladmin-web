<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <el-input v-model="query.id" clearable placeholder="输入名称或ID搜索" style="width: 200px" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker v-model="query.createTime" class="date-item" />
        <rrOperation />
      </div>
      <crudOperation :permission="permission" />
    </div>
    <!--表单组件-->
    <el-dialog append-to-body :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="570px">
      <el-form ref="form" :model="form" :rules="rules" size="small" label-width="120px">
        <el-form-item label="名称" prop="name">
          <el-input v-model="form.name" style="width: 370px" />
        </el-form-item>
        <el-form-item label="服务ID" prop="ip">
          <el-input v-model="form.ip" style="width: 370px" />
        </el-form-item>
        <el-form-item label="所属系统编号" prop="port">
          <el-input v-model="form.port"  style="width: 370px;" />
        </el-form-item>
        <el-form-item label="报文格式" prop="account">
          <el-input v-model="form.account" style="width: 370px" />
        </el-form-item>
        <el-form-item label="服务版本" prop="version">
          <el-input v-model="form.version" style="width: 370px" />
        </el-form-item>
        <el-form-item label="服务状态" prop="state">
          <el-input v-model="form.state" style="width: 370px" />
        </el-form-item>
        <el-form-item label="服务描述" prop="description">
          <el-input v-model="form.description" style="width: 370px" />
        </el-form-item>
        <el-form-item label="服务调用地址" prop="password">
          <el-input v-model="form.password" style="width: 200px" />
          <!--
          <el-button :loading="loading" type="success" style="align: right;" @click="testConnectServer">测试调用</el-button> -->
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="text" @click="crud.cancelCU">取消</el-button>
        <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
      </div>
    </el-dialog>
    <!--表格渲染-->
    <el-table ref="table" v-loading="crud.loading" :data="crud.data" style="width: 100%" @selection-change="crud.selectionChangeHandler">
      <el-table-column type="selection" width="55" />
      <el-table-column prop="name" label="名称" />
      <el-table-column prop="ip" label="ID" />
      <el-table-column prop="port" label="所属系统" />
      <el-table-column prop="account" label="报文格式" />
      <el-table-column prop="createTime" label="创建日期" />
      <el-table-column prop="version" label="服务版本" />
      <el-table-column prop="state" label="服务状态" />
      <el-table-column prop="description" label="服务描述" />
      <el-table-column v-if="checkPer(['admin','serverDeploy:edit','serverDeploy:del'])" label="操作" width="150px" align="center">
        <template slot-scope="scope">
          <udOperation
            :data="scope.row"
            :permission="permission"
          />
        </template>
      </el-table-column>
    </el-table>
    <!--分页组件-->
    <pagination />
  </div>
</template>

<script>

import crudServer from '@/api/mnt/serverDeploy'
import { testServerConnect } from '@/api/mnt/connect'
import { validateIP } from '@/utils/validate'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import DateRangePicker from '@/components/DateRangePicker'

const defaultForm = { id: null, name: null, ip: null, port: null, account: 'SOAP1.1', password: null,version:null,state:null,description:null }
export default {
  name: 'Server',
  components: { pagination, crudOperation, rrOperation, udOperation, DateRangePicker },
  cruds() {
    return CRUD({ title: '服务', url: 'api/serverDeploy', crudMethod: { ...crudServer }})
  },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  data() {
    return {
      accountList: [],
      accountMap: {},
      loading: false,
      permission: {
        add: ['admin', 'serverDeploy:add'],
        edit: ['admin', 'serverDeploy:edit'],
        del: ['admin', 'serverDeploy:del']
      },
      rules: {
        name: [
          { required: true, message: '请输入名称', trigger: 'blur' }
        ],
        ip: [
          { required: true, message: '请输入ID', trigger: 'blur' },
          { trigger: 'change' }
        ],
        port: [
          { required: true, message: '请输入系统编号', trigger: 'blur' }
        ],
        account: [
          { required: true, message: '请输入报文格式', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入调用地址', trigger: 'blur' }
        ],
        version: [
          { required: true, message: '请输入服务版本', trigger: 'blur' }
        ],
        state: [
          { required: true, message: '请输入服务状态', trigger: 'blur' }
        ],
        description: [
          { required: true, message: '请输入服务描述', trigger: 'blur' }
        ],
      }
    }
  },
  methods: {
    testConnectServer() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          this.loading = true
          testServerConnect(this.form).then((res) => {
            this.loading = false
            this.$notify({
              title: res ? '连接成功' : '连接失败',
              type: res ? 'success' : 'error',
              duration: 2500
            })
          }).catch(() => {
            this.loading = false
          })
        }
      })
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
 ::v-deep .el-input-number .el-input__inner {
    text-align: left;
  }
</style>
