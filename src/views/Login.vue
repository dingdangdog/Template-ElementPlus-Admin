<template>
  <div class="chart-container">
    <!-- set input width -->
    <div class="icon-container">
      <img src="@/assets/logo.svg" width="60" />
      <h1 style="margin-left: 2rem">System-Name</h1>
    </div>
    <div class="form-container">
      <el-form ref="loginForm" :model="formData" :rules="rules" label-width="10rem">
        <el-form-item label="帐号（Account）" prop="userName" class="login-input">
          <el-input v-model="formData.userName" />
        </el-form-item>
        <el-form-item label="密码（Password）" prop="password" class="login-input">
          <el-input
            v-model="formData.password"
            type="password"
            autocomplete="off"
            @keyup.enter="submitForm(loginForm)"
            show-password
          />
        </el-form-item>
        <el-checkbox v-model="rememberUser" class="login-button">记住我</el-checkbox>
        <!-- <el-button type="success" class="login-button" @click="toRegister">注册</el-button> -->
        <el-button type="primary" class="login-button" @click="submitForm(loginForm)"
          >登录</el-button
        >
      </el-form>
    </div>
  </div>

  <!-- 弹出框表单：注册用户 -->
  <el-dialog style="width: 25vw" v-model="registerDialog.visable" :title="registerDialog.title">
    <div class="el-dialog-main">
      <el-form ref="registerFormRef" :model="registerUser" :rules="registerUserRules">
        <el-form-item label="用户名" :label-width="formLabelWidth" prop="name">
          <el-input v-model="registerUser.name" placeholder="Name" />
        </el-form-item>

        <el-form-item label="帐号" :label-width="formLabelWidth" prop="userName">
          <el-input v-model="registerUser.userName" placeholder="Account" />
        </el-form-item>

        <el-form-item label="密码" :label-width="formLabelWidth" prop="password">
          <el-input
            v-model="registerUser.password"
            type="password"
            autocomplete="off"
            placeholder="Password"
            show-password
          />
        </el-form-item>

        <el-form-item
          label="确认密码"
          :label-width="formLabelWidth"
          prop="againPassword"
          class="is-required"
        >
          <el-input
            v-model="registerUser.againPassword"
            type="password"
            autocomplete="off"
            placeholder="Password Again"
            show-password
          />
        </el-form-item>
      </el-form>
    </div>
    <!-- 表单确认按钮 -->
    <template #footer>
      <span class="dialog-footer">
        <!-- <el-button type="primary" @click="register(registerFormRef)"> 确定 </el-button> -->
        <el-button @click="cancel()"> 取消 </el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import type { LoginUser, User } from '@/types/user'
import type { FormInstance, FormRules } from 'element-plus'
import { ElMessage } from 'element-plus'
import { ref, reactive } from 'vue'
import { login } from '@/api/api.user'
import router from '@/router/index'

// 表单输入框宽度
const formLabelWidth = ref('120px')
if (document.body.clientWidth <= 480) {
  formLabelWidth.value = '80px'
}

const loginForm = ref<FormInstance>()
const formData = ref<User>({
  userName: '',
  password: ''
})

const rememberUser = ref(true)

const rules = reactive<FormRules<typeof formData>>({
  userName: [{ required: true, message: 'Please input userName', trigger: 'blur' }],
  password: [{ required: true, message: 'Please input password', trigger: 'blur' }]
})

const submitForm = async (form: FormInstance | undefined) => {
  if (!form) {
    console.log(form)
    return
  }
  await form.validate((valid, data) => {
    if (valid) {
      //alert('submit!')
      login(rememberUser.value, formData.value).then((res: LoginUser) => {
        if (res.id != 0) {
          console.log(res)
          ElMessage({
            type: 'success',
            message: '登录成功!'
          })
          localStorage.setItem('userId', res.id.toString())
          localStorage.setItem('name', res.name)
          localStorage.setItem('token', res.token)
          // 跳转到首页
          router.push({ path: '/home' })
        } else {
          ElMessage({
            type: 'error',
            message: '登录失败!'
          })
        }
      })
    } else {
      console.log('error submit!!')
    }
  })
}

const registerDialog = ref({
  visable: false,
  title: '注册用户'
})

const registerUser = ref<User>({
  name: '',
  userName: '',
  password: '',
  againPassword: ''
})

const validatePass2 = (rule: any, value: any, callback: any) => {
  if (value === '') {
    callback(new Error('请再次输入密码'))
  } else if (value !== registerUser.value.password) {
    callback(new Error('两次输入密码不一致'))
  } else {
    callback()
  }
}

// 表单输入框校验规则
const registerUserRules = ref<FormRules>({
  name: [
    { required: true, message: '请选择用户名', trigger: 'blur' },
    { min: 1, max: 24, message: '字符限制：1-24', trigger: 'blur' }
  ],
  userName: [
    { required: true, message: '请输入登录名', trigger: 'blur' },
    { min: 6, max: 18, message: '字符限制：6-18', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 18, message: '字符限制：6-18', trigger: 'blur' }
  ],
  againPassword: [{ validator: validatePass2, trigger: 'blur' }]
})

const registerFormRef = ref<FormInstance>()

const toRegister = () => {
  registerDialog.value.visable = true
}

// const register = (form: FormInstance | undefined) => {
//   if (!form) return
//   form.validate((valid, data) => {
//     if (valid) {
//       //alert('submit!')
//       registerApi(registerUser.value).then((res) => {
//         if (res > 0) {
//           ElMessage({
//             type: 'success',
//             message: '注册成功，请登录!'
//           })
//           // 跳转到首页
//           // router.push({ path: '/' })
//           registerDialog.value.visable = false
//           formData.value.userName = registerUser.value.userName
//           formData.value.password = registerUser.value.password
//           form.resetFields()
//         } else {
//           ElMessage({
//             type: 'error',
//             message: '注册失败，请重试!'
//           })
//         }
//       })
//     } else {
//       console.log('error submit!!')
//     }
//   })
// }

const cancel = () => {
  registerDialog.value.visable = false
}
</script>

<style scoped>
.chart-container {
  padding: 2rem;
  border-radius: 10px;
  margin: 1rem;
  height: 90vh;
  border: solid 1px var(--el-menu-border-color);
  text-align: center;
  background-color: var(--el-color-primary-light-9);
}

.form-container {
  margin-top: 5rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon-container {
  margin-top: 10rem;
  /* margin-left: -3rem; */
  display: flex;
  align-items: center;
  justify-content: center;
}

.login-input {
  width: 24rem;
  /* margin-left: -3rem; */
}

.login-button {
  margin-left: 3rem !important;
  font-size: medium;
}
</style>
