<template>
  <view class="uni-container">
    <uni-forms ref="form" :value="formData" validateTrigger="bind">
      <uni-forms-item name="doctorName" label="医生名称">
        <uni-easyinput placeholder="医生名称" v-model="formData.doctorName" trim="both"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="Image" label="">
        <uni-easyinput placeholder="头像地址" v-model="formData.Image"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="Department" label="">
        <uni-easyinput placeholder="部门" v-model="formData.Department"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="Title" label="医生职位级别">
        <uni-easyinput placeholder="医生的职位" v-model="formData.Title" trim="both"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="Specialist" label="擅长方面">
        <uni-easyinput placeholder="擅长领域" v-model="formData.Specialist" trim="both"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="PersonalDetail" label="个人信息">
        <uni-easyinput placeholder="个人信息" v-model="formData.PersonalDetail" trim="both"></uni-easyinput>
      </uni-forms-item>
      <view class="uni-button-group">
        <button type="primary" class="uni-button" style="width: 100px;" @click="submit">提交</button>
        <navigator open-type="navigateBack" style="margin-left: 15px;">
          <button class="uni-button" style="width: 100px;">返回</button>
        </navigator>
      </view>
    </uni-forms>
  </view>
</template>

<script>
  import { validator } from '../../js_sdk/validator/doctorList_dushuhu.js';

  const db = uniCloud.database();
  const dbCmd = db.command;
  const dbCollectionName = 'doctorList_dushuhu';

  function getValidator(fields) {
    let result = {}
    for (let key in validator) {
      if (fields.includes(key)) {
        result[key] = validator[key]
      }
    }
    return result
  }

  export default {
    data() {
      let formData = {
        "doctorName": "",
        "Image": "头像地址",
        "Department": "",
        "Title": "",
        "Specialist": "",
        "PersonalDetail": ""
      }
      return {
        formData,
        formOptions: {},
        rules: {
          ...getValidator(Object.keys(formData))
        }
      }
    },
    onLoad(e) {
      if (e.id) {
        const id = e.id
        this.formDataId = id
        this.getDetail(id)
      }
    },
    onReady() {
      this.$refs.form.setRules(this.rules)
    },
    methods: {
      /**
       * 验证表单并提交
       */
      submit() {
        uni.showLoading({
          mask: true
        })
        this.$refs.form.validate().then((res) => {
          return this.submitForm(res)
        }).catch(() => {
        }).finally(() => {
          uni.hideLoading()
        })
      },

      /**
       * 提交表单
       */
      submitForm(value) {
        // 使用 clientDB 提交数据
        return db.collection(dbCollectionName).doc(this.formDataId).update(value).then((res) => {
          uni.showToast({
            title: '修改成功'
          })
          this.getOpenerEventChannel().emit('refreshData')
          setTimeout(() => uni.navigateBack(), 500)
        }).catch((err) => {
          uni.showModal({
            content: err.message || '请求服务失败',
            showCancel: false
          })
        })
      },

      /**
       * 获取表单数据
       * @param {Object} id
       */
      getDetail(id) {
        uni.showLoading({
          mask: true
        })
        db.collection(dbCollectionName).doc(id).field("doctorName,Image,Department,Title,Specialist,PersonalDetail").get().then((res) => {
          const data = res.result.data[0]
          if (data) {
            this.formData = data
          }
        }).catch((err) => {
          uni.showModal({
            content: err.message || '请求服务失败',
            showCancel: false
          })
        }).finally(() => {
          uni.hideLoading()
        })
      }
    }
  }
</script>
