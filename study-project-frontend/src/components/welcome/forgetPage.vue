<template>
  <div>
    <div style="margin: 30px 20px">
      <el-steps :active="active" finish-status="success" align-center>
        <el-step title="验证电子邮件"/>
        <el-step title="重新设定密码"/>
      </el-steps>
    </div>
    <transition name="el-fade-in-linear" mode="out-in">
      <div style="text-align: center;margin: 0 20px;height: 100%" v-if="active === 0">
        <div style="margin-top: 100px">
          <div style="font-size: 25px;font-weight: bold">重置密码</div>
          <div style="font-size: 14px">请输入需要重置密码的电子邮件地址</div>
        </div>
        <div style="margin-top: 50px">
          <el-form :model="form" :rules="rules" @validate="onValidate" ref="formRef">
            <el-form-item prop="email">
              <el-input v-model="form.email" type="email" placeholder="电子邮件地址">
                <template #prefix>
                  <el-icon>
                    <Message/>
                  </el-icon>
                </template>
              </el-input>
            </el-form-item>
            <el-form-item prop="verificationCode">
              <el-row :gutter="10" style="width: 100%">
                <el-col :span="17">
                  <el-input v-model="form.verificationCode" type="email" placeholder="请输入验证码" :maxlength="6">
                    <template #prefix>
                      <el-icon>
                        <EditPen/>
                      </el-icon>
                    </template>
                  </el-input>
                </el-col>
                <el-col :span="5">
                  <el-button type="success" :disabled="!isEmailValid || coldTime > 0" @click="validateEmail">
                    {{ coldTime > 0 ? '请稍后' + coldTime + '秒' : '获取验证码' }}
                  </el-button>
                </el-col>
              </el-row>
            </el-form-item>
          </el-form>

        </div>

        <div style="margin-top: 80px">
          <el-button style="width: 270px" type="danger" plain @click="startReset()">开始重置密码</el-button>
        </div>

      </div>
    </transition>

    <transition name="el-fade-in-linear" mode="out-in">
      <div style="text-align: center;margin: 0 20px;height: 100%" v-if="active === 1">
        <div style="margin-top: 100px">
          <div style="font-size: 25px;font-weight: bold">重置密码</div>
          <div style="font-size: 14px">请填写您的新密码，务必牢记，防止丢失</div>
        </div>

        <div style="margin-top: 50px">
          <el-form :model="form" :rules="rules" @validate="onValidate" ref="formRef">
            <el-form-item prop="password">
              <el-input v-model="form.password" type="password" placeholder="新密码" :maxlength="16">
                <template #prefix>
                  <el-icon>
                    <Lock/>
                  </el-icon>
                </template>
              </el-input>
            </el-form-item>
            <el-form-item prop="confirmPassword">
              <el-input v-model="form.confirmPassword" type="password" placeholder="重复新密码" :maxlength="16">
                <template #prefix>
                  <el-icon>
                    <Lock/>
                  </el-icon>
                </template>
              </el-input>
            </el-form-item>
          </el-form>
        </div>

        <div style="margin-top: 80px">
          <el-button style="width: 270px" type="danger" plain @click="doReset()">立即重置密码</el-button>
        </div>
      </div>

    </transition>
  </div>
</template>

<script setup>
import {Lock, Message, EditPen} from '@element-plus/icons-vue'
import router from "@/router/index.js";
import {reactive, ref} from "vue";
import {post} from "@/net/index.js";

const active = ref(0)

const form = reactive
({
  email: '',
  verificationCode: '',
  password: '',
  confirmPassword: ''
})


const formRef = ref()
const isEmailValid = ref(false)
const coldTime = ref(0)

const onValidate = (prop, isValid) => {
  if (prop === 'email')
    isEmailValid.value = isValid
}

const validatePassword = (rule, value, callback) => {
  if (value === '') {
    callback(new Error('请再次输入密码'))
  } else if (value !== form.password) {
    callback(new Error("两次输入的密码不一致"))
  } else {
    callback()
  }
}

const validateEmail = () => {
  coldTime.value = 60
  post('/api/auth/valid-reset-email', {
    email: form.email
  }, (message) => {
    ElMessage.success(message)

    setInterval(() => coldTime.value--, 1000)
  }, (message) => {
    ElMessage.warning(message)
    coldTime.value = 0
  })
}

const startReset = () => {
  formRef.value.validate((isValid) => {
    if (isValid) {
      post('/api/auth/start-reset', {
        email: form.email,
        verificationCode: form.verificationCode
      }, () => {
        active.value++
      })
    } else {
      ElMessage.warning('请填写电子邮件地址和验证码')
    }
  })
}

const doReset = () => {
  formRef.value.validate((isValid) => {
    if (isValid) {
      post('/api/auth/do-reset', {
        password: form.password
      }, (message) => {
        ElMessage.success(message)
        router.push('/')
      })
    } else {
      ElMessage.warning('请填写新的密码')
    }
  })
}

const rules = {
  email: [
    {required: true, message: '请输入邮件地址', trigger: 'blur'},
    {type: 'email', message: '请输入合法的电子邮件地址', trigger: ['blur', 'change']}
  ],
  verificationCode: [
    {required: true, message: '请输入获取的验证码', trigger: 'blur'}
  ],
  password: [
    {required: true, message: '请输入密码', trigger: 'blur'},
    {min: 6, max: 16, message: '密码的长度必须在6-16个字符之间', trigger: ['blur', 'change']}
  ],
  confirmPassword: [
    {validator: validatePassword, trigger: ['blur', 'change']}
  ]

}

</script>

<style scoped>

</style>