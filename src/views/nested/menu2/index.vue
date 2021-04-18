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
        <el-form-item label="流程模板名称" prop="name">
          <el-input v-model="form.name" style="width: 370px" />
        </el-form-item>
        <el-form-item label="流程编号" prop="id">
          <el-input v-model="form.id" style="width: 370px" />
        </el-form-item>
        <el-form-item label="流程分类" prop="type">
          <el-input v-model="form.type"  style="width: 370px;" />
        </el-form-item>
        <el-form-item label="流程地址" prop="url">
          <el-input v-model="form.url" style="width: 370px" />
        </el-form-item>
        <el-form-item label="流程版本号" prop="version">
          <el-input v-model="form.version" style="width: 370px" />
        </el-form-item>
        <el-form-item label="流程状态" prop="state">
          <el-input v-model="form.state" style="width: 370px" />
        </el-form-item>
        <el-form-item label="流程描述" prop="description">
          <el-input v-model="form.description" style="width: 370px" />
        </el-form-item>
        <el-form-item label="紧急程度" prop="urgent">
          <el-input v-model="form.urgent" style="width: 370px" />
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
      <el-table-column prop="name" label="流程名称" />
      <el-table-column prop="id" label="流程ID" />
      <el-table-column prop="type" label="流程分类" />
      <el-table-column prop="url" label="流程地址" />
      <el-table-column prop="version" label="流程版本号" />
      <el-table-column prop="state" label="流程状态" />
      <el-table-column prop="description" label="流程描述" />
      <el-table-column prop="urgent" label="紧急程度" />
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

  import crudServer from '@/api/mnt/process'
  import { testServerConnect } from '@/api/mnt/connect'
  import CRUD, { presenter, header, form, crud } from '@crud/crud'
  import rrOperation from '@crud/RR.operation'
  import crudOperation from '@crud/CRUD.operation'
  import udOperation from '@crud/UD.operation'
  import pagination from '@crud/Pagination'
  import DateRangePicker from '@/components/DateRangePicker'

  const defaultForm = { id: null, name: null, ip: null, port: null, account: 'SOAP1.1', password: null,version:null,state:null,description:null }
  export default {
    name: 'Process',
    components: { pagination, crudOperation, rrOperation, udOperation, DateRangePicker },
    cruds() {
      return CRUD({ title: '流程模板', url: 'api/process', crudMethod: { ...crudServer }})
    },
    mixins: [presenter(), header(), form(defaultForm), crud()],
    data() {
      return {
        accountList: [],
        accountMap: {},
        loading: false,
        permission: {
          add: ['admin', 'process:add'],
          edit: ['admin', 'process:edit'],
          del: ['admin', 'process:del']
        },
        rules: {
          name: [
            { required: true, message: '请输入名称', trigger: 'blur' }
          ],
          id: [
            { required: true, message: '请输入ID', trigger: 'blur' },
            { trigger: 'change' }
          ],
          type: [
            { required: true, message: '请输入流程模板类型', trigger: 'blur' }
          ],
          url: [
            { required: true, message: '请输入流程模板地址', trigger: 'blur' }
          ],
          version: [
            { required: true, message: '请输入流程模板版本', trigger: 'blur' }
          ],
          state: [
            { required: true, message: '请输入流程模板状态', trigger: 'blur' }
          ],
          description: [
            { required: true, message: '请输入流程模板描述', trigger: 'blur' }
          ],
          urgent: [
            { required: true, message: '请输入流程模板紧急程度', trigger: 'blur' }
          ],
        }
      }
    },

  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  ::v-deep .el-input-number .el-input__inner {
    text-align: left;
  }
</style>

