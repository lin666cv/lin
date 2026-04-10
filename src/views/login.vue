<script setup>
import { Back } from '@element-plus/icons-vue';
import { ref } from 'vue'
import { login } from '@/api/admin'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
const router = useRouter()
const ruleFormRef = ref()
const submitForm =()=> {
  ruleFormRef.value.validate((valid) => {
    if (valid) {
     login(formData.value).then(data=>{
      console.log(data)
      if(!data.data.data.token){
        return console.error('登录失败')
      }
      localStorage.setItem('token', data.data.data.token)
      localStorage.setItem('userInfo', JSON.stringify(data.data.data.userInfo))
      if(data.data.data.userInfo.userType === 2){
        router.push('/back/dashboard')
        ElMessage.success('登录成功')
      }else if(data.data.data.userInfo.userType === 1){
        router.push('/home')
        ElMessage.success('登录成功')
      }
     })
    } else {
      console.log('表单验证失败')
    }
  })
}
const formData = ref({
  username: '',
  password: ''
})
const rules = ref({
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' }
  ]
})
</script>

<template>
  <div class="container">
    <div class="title">
<div class="back-home">
  <el-icon class="icon-arrow-left">
    <Back />
  </el-icon>
  <span @click="router.push('/home')">返回首页</span>
</div>
<div class="title-text">
  <h2>登录您的账户</h2>
  <p>请输入您的登录信息</p>
</div>
    </div>
    <div class="form-container">
      <el-form
          ref="ruleFormRef"
          :model="formData"
          :rules="rules"
          label-position="top"
        >
          <el-form-item label="用户名/邮箱" prop="username">
            <el-input v-model="formData.username" size="large" placeholder="请输入用户名" />
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="formData.password" placeholder="请输入密码" size="large" type="password" show-password />
          </el-form-item>
          <el-button class="btn" size="large" type="primary" @click="submitForm(ruleFormRef)">登录</el-button>
      </el-form>
      <div class="footer">
            <p >还没账号？<router-link type="primary" to="register">去注册</router-link></p>
          </div>
    </div>
  </div>
</template>
<style scoped lang="scss">
 .container {
  width: 384px;
 .title {
  
 .back-home {
  margin-bottom: 60px;
 }
 .title-text {
  text-align: center;
  h2 {
    font-size: 36px;
   margin-bottom: 10px;
  }
  p {
    font-size: 18px;
    color: #6b7280;
  }
 }
}
.form-container {
  margin-top: 30px;
  .btn {
    margin-top: 40px;
    width: 100%;
  }
  .footer {
    margin-top: 20px;
    text-align: center;
  }
  }
}
</style>