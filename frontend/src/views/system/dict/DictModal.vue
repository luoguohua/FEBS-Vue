<template>
  <a-modal
    :title="title"
    :width="600"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="字典名称">
          <a-input placeholder="请输入字典名称" v-decorator="[ 'dictName', validatorRules.dictName]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="字典编码">
          <a-input placeholder="请输入字典编码" v-decorator="[ 'dictCode', validatorRules.dictCode]"/>
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="描述">
          <a-input v-decorator="[ 'description']"/>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import pick from 'lodash.pick'
import {addObj, editObj} from '@/api/sys/dict'

export default {
  name: 'DictModal',
  data () {
    return {
      value: 1,
      title: '操作',
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
      form: this.$form.createForm(this),
      validatorRules: {
        dictName: {rules: [{ required: true, message: '请输入字典名称!' }]},
        dictCode: {rules: [{ required: true, message: '请输入字典编码!' }]}
      }
    }
  },
  created () {
  },
  methods: {
    handleChange (value) {
      this.model.status = value
    },
    add () {
      this.edit({})
    },
    edit (record) {
      if (record.id) {
        this.visiblekey = true
      } else {
        this.visiblekey = false
      }
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'dictName', 'dictCode', 'description'))
      })
    },
    // 确定
    handleOk () {
      const that = this
      // 触发表单验证
      this.form.validateFields((err, values) => {
        if (!err) {
          that.confirmLoading = true
          let formData = Object.assign(this.model, values)
          let obj
          if (!this.model.id) {
            formData.delFlag = '1'
            obj = addObj(formData)
          } else {
            obj = editObj(formData)
          }
          obj.then((response) => {
            const res = response.data
            if (res.code === 1) {
              that.$message.success('添加成功')
              that.$emit('ok')
            } else {
              that.$message.warning('添加失败')
            }
          }).finally(() => {
            that.confirmLoading = false
            that.close()
          })
        }
      })
    },
    // 关闭
    handleCancel () {
      this.close()
    },
    close () {
      this.$emit('close')
      this.visible = false
    }
  }
}
</script>
