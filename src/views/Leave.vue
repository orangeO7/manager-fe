<template>
  <div class="leave-manager">
    <div class="query-form">
      <el-form ref="form" :model="queryForm" :inline="true">
        <el-form-item label="审批状态" prop="applyState">
          <el-select v-model="queryForm.applyState">
            <el-option value="" label="全部"></el-option>
            <el-option :value="1" label="待审批"></el-option>
            <el-option :value="2" label="审批中"></el-option>
            <el-option :value="3" label="审批拒绝"></el-option>
            <el-option :value="4" label="审批通过"></el-option>
            <el-option :value="5" label="作废"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getApplyList">查询</el-button>
          <el-button @click="handleReset('form')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="base-table">
      <div class="action">
        <el-button type="primary" @click="handleApply">申请休假</el-button>
      </div>
      <el-table :data="applyList">
        <el-table-column
          v-for="item in columns"
          :key="item.prop"
          :prop="item.prop"
          :label="item.label"
          :width="item.width"
          :formatter="item.formatter"
        ></el-table-column>
        <el-table-column label="操作" width="150">
          <template #default="scope">
            <el-button size="mini" @click="handleDetail(scope.row)">查看</el-button>
            <el-button
              v-if="[1, 2].includes(scope.row.applyState)"
              type="danger"
              size="mini"
              @click="handleDelete(scope.row._id)"
              >作废</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        class="pagination"
        background
        layout="prev, pager, next"
        :total="pager.total"
        :page-size="pager.pageSize"
        @current-change="handleCurrentChange"
      />
    </div>
    <el-dialog v-model="showModal" title="申请休假" width="70%">
      <el-form
        ref="dialogForm"
        :model="leaveForm"
        label-width="120px"
        :rules="rules"
      >
        <el-form-item label="休假类型" prop="applyType" required>
          <el-select v-model="leaveForm.applyType">
            <el-option label="事假" :value="1"></el-option>
            <el-option label="调休" :value="2"></el-option>
            <el-option label="年假" :value="3"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="休假时间" required>
          <el-row>
            <el-col :span="8">
              <el-form-item prop="startTime">
                <el-date-picker
                  v-model="leaveForm.startTime"
                  type="date"
                  placeholder="选择开始日期"
                  @change="(val) => handleDateChange('startTime', val)"
                />
              </el-form-item>
            </el-col>
            <el-col :span="1">-</el-col>
            <el-col :span="8">
              <el-form-item prop="endTime">
                <el-date-picker
                  v-model="leaveForm.endTime"
                  type="date"
                  placeholder="选择结束日期"
                  @change="(val) => handleDateChange('endTime', val)"
                />
              </el-form-item>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="休假时长" prop="leaveTime" required>
          {{ leaveForm.leaveTime }}
        </el-form-item>
        <el-form-item label="休假原因" prop="reasons">
          <el-input
            v-model="leaveForm.reasons"
            type="textarea"
            row="3"
            placeholder="请输入休假原因"
          />
        </el-form-item>
      </el-form>
      <template #footer>
          <el-button size="mini" @click="handleClose">取 消</el-button>
          <el-button type="primary" size="mini" @click="handleSubmit">确 定</el-button>
      </template>
    </el-dialog>
    <el-dialog
      v-model="showDetailModal"
      title="休假详情"
      destroy-on-close
    >
      <el-steps
        :active="detail.applyState > 2 ? 3 : detail.applyState"
        align-center
      >
        <el-step title="待审批"></el-step>
        <el-step title="审批中"></el-step>
        <el-step title="审批通过/审批拒绝"></el-step>
      </el-steps>
      <el-form label-width="120px" label-suffix=":">
        <el-form-item label="休假类型">
          {{ detail.applyTypeName }}
        </el-form-item>
        <el-form-item label="休假时间">
          {{ detail.time }}
        </el-form-item>
        <el-form-item label="休假时长">
          {{ detail.leaveTime }}
        </el-form-item>
        <el-form-item label="休假原因">
          {{ detail.reasons }}
        </el-form-item>
        <el-form-item label="审批状态">
          {{ detail.applyStateName }}
        </el-form-item>
        <el-form-item label="审批人">
          {{ detail.curAuditUserName }}
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import { ref, reactive, onMounted, getCurrentInstance } from 'vue'
import utils from '../utils/utils'

export default {
  name: 'Leave',
  setup() {
    const { proxy } = getCurrentInstance()
    const queryForm = reactive({
      applyState: 1
    })
    const pager = reactive({
      pageNum: 1,
      pageSize: 10,
      total: 0
    })
    const columns = reactive([
      {
        label: '单号',
        prop: 'orderNo'
      },
      {
        label: '休假时间',
        prop: '',
        formatter(row) {
          return (
            utils.formatDate(new Date(row.startTime), 'yyyy-MM-dd') +
            '到' +
            utils.formatDate(new Date(row.endTime), 'yyyy-MM-dd')
          )
        }
      },
      {
        label: '休假时长',
        prop: 'leaveTime'
      },
      {
        label: '休假类型',
        prop: 'applyType',
        formatter(row, column, value) {
          return {
            1: '事假',
            2: '调休',
            3: '年假'
          }[value]
        }
      },
      {
        label: '休假原因',
        prop: 'reasons'
      },
      {
        label: '申请时间',
        prop: 'createTime',
        width: 180,
        formatter(row, column, value) {
          return utils.formatDate(new Date(value))
        }
      },
      {
        label: '审批人',
        prop: 'auditUsers'
      },
      {
        label: '当前审批人',
        prop: 'curAuditUserName'
      },
      {
        label: '审批状态',
        prop: 'applyState',
        formatter(row, column, value) {
          return {
            1: '待审批',
            2: '审批中',
            3: '审批拒绝',
            4: '审批通过',
            5: '作废'
          }[value]
        }
      }
    ])
    const applyList = ref([])
    const leaveForm = reactive({
      applyType: 1,
      startTime: '',
      endTime: '',
      leaveTime: '0天',
      reasons: ''
    })
    // create: 创建 delete: 作废
    const action = ref('create')
    const showModal = ref(false)
    const showDetailModal = ref(false)
    const detail = ref({})
    const rules = {
      startTime: [
        {
          type: 'date',
          required: true,
          message: '请输入开始日期',
          trigger: 'change'
        }
      ],
      endTime: [
        {
          type: 'date',
          required: true,
          message: '请输入结束日期',
          trigger: 'change'
        }
      ],
      reasons: [
        {
          required: true,
          message: '请输入休假原因',
          trigger: ['change', 'blur']
        }
      ]
    }
    // 初始化接口调用
    onMounted(() => {
      getApplyList()
    })
    // 获取申请列表
    const getApplyList = async () => {
      try {
        const params = { ...queryForm, ...pager }
        const { list, page } = await proxy.$api.getApplyList(params)
        applyList.value = list
        pager.total = page.total
      } catch(error) {
        console.error('getApplyList error:', error)
      }
    }
    // 重置查询表单
    const handleReset = (form) => {
      proxy.$refs[form].resetFields()
    }
    // 分页事件处理
    const handleCurrentChange = (current) => {
      pager.pageNum = current
      getApplyList()
    }
    // 点击申请休假-展示弹框
    const handleApply = () => {
      showModal.value = true
      action.value = 'create'
    }
    // 弹框关闭
    const handleClose = () => {
      showModal.value = false
      handleReset('dialogForm')
    }
    // 获取休假时长
    const handleDateChange = (key, val) => {
      let { startTime, endTime } = leaveForm
      if (!startTime || !endTime) return
      if (startTime > endTime) {
        proxy.$message.error('开始日期不能晚于结束日期')
        leaveForm.leaveTime = '0天'
        setTimeout(() => {
          leaveForm[key] = ''
        }, 0)
      } else {
        leaveForm.leaveTime = (endTime - startTime) / (24 * 60 * 60 * 1000) + 1 + '天'
      }
    }
    // 申请提交
    const handleSubmit = () => {
      proxy.$refs.dialogForm.validate(async (valid) => {
        if (valid) {
          try {
            let params = { ...leaveForm, action: action.value }
            await proxy.$api.leaveOperate(params)
            proxy.$message.success('创建成功')
            handleClose()
            getApplyList()
          } catch (error) {
            console.error('handleSubmit error:', error)
          }
        }
      })
    }
    const handleDetail = (row) => {
      let data = {
        ...row
      }
      data.applyTypeName = {
        1: '事假',
        2: '调休',
        3: '年假'
      }[row.applyType]
      data.time = (
        utils.formatDate(new Date(row.startTime), 'yyyy-MM-dd') +
        '到' +
        utils.formatDate(new Date(row.endTime), 'yyyy-MM-dd')
      )
      data.applyStateName = {
        1: '待审批',
        2: '审批中',
        3: '审批拒绝',
        4: '审批通过',
        5: '作废'
      }[row.applyState]
      detail.value = data
      showDetailModal.value = true
    }
    const handleDelete = async (_id) => {
      try {
        let params = { _id, action: 'delete' }
        await proxy.$api.leaveOperate(params)
        proxy.$message.success('删除成功')
        getApplyList()
      } catch (error) {
        console.error('handleDelete error:', error)
      }
    }
    return {
      queryForm,
      pager,
      columns,
      applyList,
      leaveForm,
      showModal,
      rules,
      showDetailModal,
      detail,
      getApplyList,
      handleReset,
      handleCurrentChange,
      handleApply,
      handleClose,
      handleSubmit,
      handleDateChange,
      handleDetail,
      handleDelete
    }
  }
}
</script>

<style lang="scss">
</style>
