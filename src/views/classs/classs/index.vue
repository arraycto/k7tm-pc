<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <el-input
          v-model="query.name"
          clearable
          placeholder="输入名称搜索"
          style="width: 134px"
          class="filter-item"
          @keyup.enter.native="crud.toQuery"
        />
        <el-select
          v-model="query.type"
          clearable
          placeholder="班级类型搜索"
          class="filter-item"
          style="width: 134px"
          @change="crud.toQuery"
        >
          <el-option
            v-for="(item, index) in dict.classs_type"
            :key="item.label + index"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
        <label class="el-form-item-label">状态</label>
        <el-input v-model="query.status" clearable placeholder="状态" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">课程类型</label>
        <el-input v-model="query.courseId" clearable placeholder="课程类型" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">职业顾问</label>
        <el-input v-model="query.vcTeacher" clearable placeholder="职业顾问" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">技术顾问</label>
        <el-input v-model="query.taTeacher" clearable placeholder="技术顾问" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">项目经理</label>
        <el-input v-model="query.pmTeacher" clearable placeholder="项目经理" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.classTime"
          start-placeholder="classTimeStart"
          end-placeholder="classTimeStart"
          class="date-item"
        />
        <date-range-picker
          v-model="query.graduateTime"
          start-placeholder="graduateTimeStart"
          end-placeholder="graduateTimeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" :width="dialogStyle">
        <el-form ref="form" :inline="true" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="名称" prop="name">
            <el-input v-model="form.name" style="width: 194px;" />
          </el-form-item>
          <el-form-item label="类型" prop="type">
            <el-select v-model="form.type" clearable style="width: 194px">
              <el-option
                v-for="(item, index) in dict.classs_type"
                :key="item.label + index"
                :label="item.label"
                :value="parseInt(item.value)"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="状态">
            未设置字典，请手动设置 Select
          </el-form-item>
          <el-form-item label="课程类型">
            未设置字典，请手动设置 Select
          </el-form-item>
          <el-form-item label="职业顾问" prop="vcTeacher">
            未设置字典，请手动设置 Select
          </el-form-item>
          <el-form-item label="技术顾问">
            未设置字典，请手动设置 Select
          </el-form-item>
          <el-form-item label="项目经理">
            未设置字典，请手动设置 Select
          </el-form-item>
          <el-form-item label="开班时间">
            <el-date-picker v-model="form.classTime" type="datetime" style="width: 194px;" />
          </el-form-item>
          <el-form-item label="毕业时间">
            <el-date-picker v-model="form.graduateTime" type="datetime" style="width: 194px;" />
          </el-form-item>
          <br>
          <el-form-item label="班级描述">
            <el-input
              v-model="form.desc"
              type="textarea"
              :autosize="{minRows:3}"
              placeholder="班级的情况都可以写在这里"
              :style="'width:'+textareaStyle"/>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer" align="center">
          <el-button @click="crud.cancelCU">关闭</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="name" label="名称" align="center"/>
        <el-table-column prop="type" label="类型" align="center">
          <template slot-scope="scope">
            {{ dict.label.classs_type[scope.row.type] }}
          </template>
        </el-table-column>
        <el-table-column prop="status" label="状态" />
        <el-table-column prop="stuNum" label="现有人数" />
        <el-table-column prop="classTransferNum" label="转班人数" />
        <el-table-column prop="temporaryAbsenceNum" label="休学人数" />
        <el-table-column prop="refundNum" label="退学人数" />
        <el-table-column prop="expelNum" label="开除人数" />
        <el-table-column prop="courseId" label="课程类型" />
        <el-table-column prop="vcTeacher" label="职业顾问" />
        <el-table-column prop="taTeacher" label="技术顾问" />
        <el-table-column prop="pmTeacher" label="项目经理" />
        <el-table-column prop="classTime" label="开班时间" width="120" :formatter="formatDate" align="center" />
        <el-table-column prop="graduateTime" label="毕业时间" width="120" :formatter="formatDate" align="center">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.graduateTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="updateBy" label="更新者" />
        <el-table-column prop="updateTime" label="更新时间" width="135" align="center">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.updateTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','classs:edit','classs:del']" label="操作" width="150px" align="center">
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
  </div>
</template>

<script>
import crudClasss from '@/api/classs/classs'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import { checkDevice } from '@/utils/index'

const defaultForm = { classId: null, name: null, type: null, status: null, stuNum: null, classTransferNum: null, temporaryAbsenceNum: null, refundNum: null, expelNum: null, courseId: null, vcTeacher: null, taTeacher: null, pmTeacher: null, classTime: null, graduateTime: null, desc: null, createBy: null, updateBy: null, createTime: null, updateTime: null}
export default {
  name: 'Classs',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['classs_type'],
  cruds() {
    return CRUD({ title: '班级', url: 'api/classs', sort: 'classId,desc', crudMethod: { ...crudClasss }})
  },
  data() {
    return {
      textareaStyle: '270%', dialogStyle: '660px',
      permission: {
        add: ['admin', 'classs:add'],
        edit: ['admin', 'classs:edit'],
        del: ['admin', 'classs:del']
      },
      rules: {
        name: [
          { required: true, message: '名称不能为空', trigger: 'blur' }
        ],
        type: [
          { required: true, message: '请选择类型', trigger: 'blur' }
        ],
        vcTeacher: [
          { required: true, message: '职业顾问不能为空', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '名称不能为空', trigger: 'blur' }
        ],
        vcTeacher: [
          { required: true, message: '职业顾问不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'name', display_name: '名称' },
        { key: 'type', display_name: '类型' },
        { key: 'status', display_name: '状态' },
        { key: 'courseId', display_name: '课程类型' },
        { key: 'vcTeacher', display_name: '职业顾问' },
        { key: 'taTeacher', display_name: '技术顾问' },
        { key: 'pmTeacher', display_name: '项目经理' }
      ]
    }
  },
  created() {
    // 如果是手机/平板端 调整显示
    if (this.checkDevice().isPhone) {
      // 弹框100%  文本域100%
      this.textareaStyle = '108%'
      this.dialogStyle = '100%'
    } else if (this.checkDevice().isTablet) {
      this.textareaStyle = '270%'
    }
  },
  methods: {
    checkDevice,
    formatDate(row, column) {
      // 获取单元格数据
      const rowData = row[column.property]
      const dt = new Date(rowData)
      return dt.getFullYear() + '-' + (dt.getMonth() + 1) + '-' + dt.getDate()
    },
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
