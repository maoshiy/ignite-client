<template>
  <div class="nodes g_wrap">
    <el-button @click="createNodeVis = true" type="primary">创建节点</el-button>
      <t-c-r
        :tableData="tableData"
        :tableCols="tableCols"
      >
        <div slot="operator" slot-scope="scope">
          <el-dropdown>
            <el-button type="primary" size="mini">
              操作<i class="el-icon-arrow-down el-icon--right"></i>
            </el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="editNode(scope.row)">编辑</el-dropdown-item>
              <el-dropdown-item @click.native="handleDelete(scope.row)">删除</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </div>
      </t-c-r>
      <el-dialog
        :visible.sync="createNodeVis"
        @close="handleClose"
        title="创建节点"
        width="600px">
        <el-form label-width="150px" :model="form" :rules="rules" ref="form">
          <el-form-item label="节点名称" prop="name">
            <el-input v-model="form.name"></el-input>
          </el-form-item>
          <el-form-item label="address(agent)" prop="address">
            <el-input v-model="form.address"></el-input>
          </el-form-item>
          <el-form-item label="connection(ignite)" prop="connection">
            <el-input v-model="form.connection"></el-input>
          </el-form-item>
          <el-form-item label="端口范围">
            <el-col :span="6">
              <el-input v-model.number="form.port_from" placeholder="起始端口"></el-input>
            </el-col>
            <el-col class="line" :span="2" style="text-align: center;">-</el-col>
            <el-col :span="6">
              <el-input v-model.number="form.port_to" placeholder="结束端口"></el-input>
            </el-col>
          </el-form-item>
          <el-form-item label="备注">
            <el-input v-model="form.comment" type="textarea"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <template v-if="isEdit">
            <el-button type="primary" @click="handleUpdate">修改</el-button>
          </template>
          <template v-else>
            <el-button type="primary" @click="handleCreateDialog">创建</el-button>
          </template>
        </div>
      </el-dialog>
  </div>
</template>

<script>
import request from '@/apis/request'
import TCR from '@/components/TableColumnRender.vue'

const FormInit = {
  name: '',
  address: '',
  connection: '',
  port_from: '',
  port_to: '',
  comment: '',
}

const rules = {
  name: [{ required: true, message: '请输入名称', trigger: 'blur' }],
  address: [{ required: true, message: '请输入address', trigger: 'blur' }],
  connection: [{ required: true, message: '请输入connection', trigger: 'blur' }],
  // port: [{ type: 'date', required: true, message: '请选择时间', trigger: 'change' }],
}

export default {
  components: {
    TCR,
  },

  data() {
    return {
      form: { ...FormInit },
      rules,
      tableData: [],
      createNodeVis: false,
      isEdit: false,
      editId: null,
    }
  },

  computed: {
    tableCols() {
      return [
        {
          raw: {
            label: '节点名称',
            prop: 'name',
          },
        },
        {
          raw: {
            label: 'agent地址',
            prop: 'address',
          },
        },
        {
          raw: {
            label: 'ignite地址',
            prop: 'connection',
          },
        },
        {
          raw: {
            label: '端口范围',
          },
          formatter: (data, row) => `${row.port_from} ~ ${row.port_to}`,
        },
        {
          raw: {
            label: '备注',
            prop: 'comment',
          },
        },
        {
          raw: {
            label: '操作',
          },
          slot: 'operator',
        },
      ]
    },
  },

  methods: {

    fetchNodes(page = 1) {
      request
        .get('/api/admin/auth/nodes')
        .then((response) => {
          if (!response) return
          this.tableData = response
        })
    },

    async handleCreateDialog() {
      await this.$refs.form.validate()
      // Vaidate Dialog
      request.post('/api/admin/auth/nodes', this.form).then(() => {
        this.$message('创建成功')
        this.fetchNodes()
        this.createNodeVis = false
      })
    },

    handleDelete({ id }) {
      request.delete(`/api/admin/auth/nodes/${id}`).then(() => {
        this.$message('删除成功')
        this.createNodeVis = false
        this.fetchNodes()
      })
    },

    async handleUpdate() {
      await this.$refs.form.validate()
      request.put(`/api/admin/auth/nodes/${this.editId}`, this.form).then(() => {
        this.$message('更新成功')
        this.createNodeVis = false
        this.fetchNodes()
      })
    },

    editNode(row) {
      Object.keys(this.form).forEach((key) => {
        this.form[key] = row[key]
      })
      this.editId = row.id
      this.isEdit = true
      this.createNodeVis = true
    },

    handleClose() {
      this.form = { ...FormInit }
      this.isEdit = false
      this.editId = null
    },
  },

  async created() {
    this.fetchNodes()
  },
}
</script>

<style lang="less" scoped>
</style>
