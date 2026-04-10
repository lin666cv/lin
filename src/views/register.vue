<template>
  <div class="container">
    <div class="title">
     <div class="title-text">
        <h2>创建你的账号</h2>
        <p>请输入你的账号信息</p>
     </div>
    </div>
    <div class="form-container">
      <el-form :model="formData" :rules="rules" ref="submitFormRef" label-width="120px" label-position="top">
        <el-form-item label="用户名或邮箱" prop="username">
          <el-input v-model="formData.username" placeholder="请输入用户名" size="large"/>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="formData.email" placeholder="请输入邮箱" size="large"/>
        </el-form-item>
        <el-form-item label="昵称" prop="nickname">
          <el-input v-model="formData.nickname" placeholder="请输入昵称" size="large"/>
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="formData.phone" placeholder="请输入手机号" size="large"/>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="formData.password" type="password" placeholder="请输入密码" size="large" show-password/>
        </el-form-item>
        <el-form-item label="确认密码" prop="confirmPassword">
          <el-input v-model="formData.confirmPassword" type="password" placeholder="请输入确认密码" size="large" show-password/>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm(submitFormRef)" class="btn">注册</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
  </template>
<script setup>
import { ref,reactive } from 'vue'
import { ElMessage } from 'element-plus'
import { register } from '@/api/frontend'
import { useRouter } from 'vue-router'
const formData = ref({
    "username": "",
    "email": "",
    "nickname": "",
    "phone": "",
    "password": "",
    "confirmPassword": "",
    "gender": 0,
    "userType": 1
})
const rules = ref({
    username: [
        { required: true, message: '请输入用户名', trigger: 'blur' }
    ],
    password: [
        { required: true, message: '请输入密码', trigger: 'blur' }
    ],
    confirmPassword: [
        { required: true, message: '请确认密码', trigger: 'blur' }
    ],
    email: [
        { required: true, message: '请输入邮箱', trigger: 'blur' }
    ]
})
const router = useRouter()
const submitFormRef = ref(null)
const submitForm = async (formel) => {
  if ( !formel) return
   formel.validate(async (valid) => {
    register(formData.value).then(({data})=>{
      if(data.code === "BUSINESS_ERROR") {
        ElMessage.error(data.message)
      }else{
        ElMessage.success('注册成功')
        router.push('/auth/login')
      }
    })
  })
}
</script>
<style scoped lang="scss">
.container {
    width: 384px;
    .flex-box {
        display: flex;
        align-items: center;
    }
    .title {
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
            padding: 30px;
            text-align: center;
        }
    }
}
</style>