<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    :destroyOnClose="true"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item label="父级节点" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-tree-select
            ref="treeSelect"
            placeholder="请选择父级节点"
            v-decorator="['parentnode', validatorRules.parentnode]"
            :trigger-change="true"
            dict="pmp_project,taskname,id"
            pidField="parentnode"
            pidValue="0"
          ></j-tree-select>
        </a-form-item>
        <a-form-item label="任务名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input
            v-decorator="[ 'taskname', validatorRules.taskname]"
            placeholder="请输入任务名称"
            :trigger-change="true"
          ></a-input>
        </a-form-item>
        <!--<a-form-item label="类型编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'projectcode', validatorRules.projectcode]" placeholder="请输入类型编码"></a-input>
        </a-form-item>-->

        <!--<a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol">
          <span style="font-size: 12px;color:red" slot="label">编码规则(注)</span>
          <span style="font-size: 12px;color:red">
            编码值前缀需和父节点保持一致,比如父级节点编码是A01则当前编码必须以A01开头
          </span>
        </a-form-item>-->
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import { validateDuplicateValue } from '@/utils/util'
import { httpAction, getAction } from '@/api/manage'
import pick from 'lodash.pick'
import JTreeSelect from '@/components/jeecg/JTreeSelect'

export default {
  name: 'SysCategoryModal',
  components: {
    JTreeSelect
  },
  data() {
    return {
      form: this.$form.createForm(this),
      title: '操作',
      width: 800,
      visible: false,
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },

      confirmLoading: false,
      validatorRules: {
        parentnode: { rules: [{ required: true, message: '请选择父级节点!' }] },
        taskname: {
          rules: [
            { required: true, message: '请输入任务名称!' },
          ]
        },
        projectcode: {
          rules: [
            {
              required: true,
              message: '请输入任务编码!'
            },
            {
              validator: this.validateMyCode
            }
          ]
        }
      },
      url: {
        add: '/protree/pmpProject/addTask',
        edit: '/protree/pmpProject/edit',
        checkCode: '/protree/pmpProject/checkCode'
      },
      expandedRowKeys: [],
      pidField: 'parentnode'
    }
  },
  created() {},
  methods: {
    add() {
      this.edit({})
    },
    edit(record) {
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'parentnode', 'taskname', 'projectcode'))
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
    },
    handleOk() {
      const that = this
      // 触发表单验证
      this.form.validateFields((err, values) => {
        if (!err) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          let params = {
            projectname: this.$route.query.data.projectname
          }
          let formData = Object.assign(this.model, values, params)
          console.log('表单提交数据', formData)
          httpAction(httpurl, formData, method)
            .then(res => {
              if (res.success) {
                that.$message.success(res.message)
                that.submitSuccess(formData)
              } else {
                that.$message.warning(res.message)
              }
            })
            .finally(() => {
              that.confirmLoading = false
              that.close()
            })
        }
      })
    },
    handleCancel() {
      this.close()
    },
    popupCallback(row) {
      this.form.setFieldsValue(pick(row, 'parentnode', 'taskname', 'projectcode'))
    },
    submitSuccess(formData) {
      if (!formData.id) {
        let treeData = this.$refs.treeSelect.getCurrTreeData()
        this.expandedRowKeys = []
        this.getExpandKeysByPid(formData[this.pidField], treeData, treeData)
        this.$emit('ok', formData, this.expandedRowKeys.reverse())
      } else {
        this.$emit('ok', formData)
      }
    },
    getExpandKeysByPid(pid, arr, all) {
      if (pid && arr && arr.length > 0) {
        for (let i = 0; i < arr.length; i++) {
          if (arr[i].key == pid) {
            this.expandedRowKeys.push(arr[i].key)
            this.getExpandKeysByPid(arr[i]['parentId'], all, all)
          } else {
            this.getExpandKeysByPid(pid, arr[i].children, all)
          }
        }
      }
    },
    validateMyCode(rule, value, callback) {
      let params = {
        parentnode: this.form.getFieldValue('parentnode'),
        projectcode: value
      }
      console.log(params)
      getAction(this.url.checkCode, params).then(res => {
        if (res.success) {
          callback()
        } else {
          callback(res.message)
        }
      })
    }
  }
}
</script>