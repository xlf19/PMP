<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input v-if="showInput" :readOnly="true" v-decorator="[ 'taskName', validatorRules.taskName]" placeholder="请输入任务名称"></a-input>
            <j-tree-select v-else
              v-decorator="[ 'taskid', validatorRules.taskid]"
              placeholder="请选择菜单"
              dict="pmp_project,projectname,id"
              pidField="parentnode"
              pidValue="0"/>
        </a-form-item>

        <a-form-item label="任务小结" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-editor v-decorator="['content',{trigger:'input'}]"/>
        </a-form-item>
        <a-form-item label="附件" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['contentAnnex', validatorRules.contentAnnex]" :trigger-change="true"></j-upload>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import pick from 'lodash.pick'
  import { httpAction } from '@/api/manage'
  import JUpload from '@/components/jeecg/JUpload'
  import JEditor from '@/components/jeecg/JEditor'
  import { validateDuplicateValue } from '@/utils/util'
  import JTreeSelect from '@/components/jeecg/JTreeSelect'

  export default {
    name: "PmpTaskSummaryModal",
    components: { 
      JUpload,
      JEditor,
      JTreeSelect,
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        model: {},
        showInput: false,
        labelCol: {
          xs: { span: 24 },
          sm: { span: 2 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 22 },
        },
        confirmLoading: false,
        validatorRules: {
          taskid: {rules: [
            {required: true, message: '请输入任务id!'},
          ]},
          content: {rules: [
            {required: true, message: '请输入任务小结!'},
          ]},
          contentAnnex: {rules: [
          ]},
        },
        url: {
          add: "/summary/pmpTaskSummary/add",
          edit: "/summary/pmpTaskSummary/edit",
        }
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        console.log(record);

        if ("undefined" == typeof(record.id))
          this.showInput = false;
        else
          this.showInput = true;

        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'id','taskid','taskName','content','contentAnnex'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
         
        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'taskid','content','contentAnnex'))
      },

      
    }
  }
</script>