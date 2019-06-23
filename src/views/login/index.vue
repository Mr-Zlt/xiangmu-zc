<template>
  <div class="login-wrap">
    <div class="login-from-wrap">
    <div class="login-head"><img src="./logo_index.png" alt=""></div>
    <div class="login-form">
        <el-form  :model="form" :rules="rules" ref="ruleForm"  >
      <el-form-item prop="mobile">
        <el-input v-model="form.mobile" placeholder="手机号"></el-input>
      </el-form-item>
        <el-form-item prop="code">
        <el-col :span="10"> <el-input v-model="form.code" placeholder="验证码"> </el-input>
          </el-col>
        <el-col :span="10"><el-button @click="handSendCode">获取验证码</el-button>
        </el-col>
      </el-form-item>
       <el-from-item>
          <el-checkbox v-model="form.agree"></el-checkbox>
          <span>我已阅读并同意<a href="#">用户协议</a>和 <a href="#">隐私条款</a></span>
        </el-from-item>
      <el-form-item>
        <el-button
         class="btn-login"
         type="primary"
         @click="handleLogin"
         :loading="loginLoading"
         >登陆</el-button>
        </el-form-item>
    </el-form>
    </div>
  </div>
    </div>
</template>

<script>
import axios from 'axios'
import '@/vendor/gt'
export default {
  name: 'AppLogin',
  data () {
    return {
      form: {
        mobile: '',
        code: '',
        agree: ''
      },
      loginLoading: false,
      rules: {
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { len: 11, max: 11, message: '长度在 1 到 1 个字符', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入验证码', trigger: 'blur' },
          { len: 6, max: 6, message: '长度在 6 到 6 个字符', trigger: 'blur' }
        ],
        agree: [
          { required: true, message: '请同意用户协议', trigger: 'change' },
          { pattern: /true/, message: '请同意用户协议', trigger: 'change' }
        ]
      },
      captchaObj: null
    }
  },
  methods: {
    handleLogin () {
      this.$refs[ 'ruleForm' ].validate(valid => {
        if (!valid) {
          return
        }
        this.submitLogin()
      })
    },
    submitLogin () {
      this.loginLoading = true
      axios({
        method: 'POST',
        url: 'http://ttapi.research.itcast.cn/mp/v1_0/authorizations',
        data: this.form
      }).then(res => {
        this.$message({
          message: '恭喜你，这是一条成功消息',
          type: 'success'
        })
        this.loginLoading = false
        this.$router.push({
          name: 'home'
        })
      }).catch(err => {
        if (err.response.status === 400) {
          this.$message.error('登陆失败，手机或验证码错误')
          this.loginLoading = false
        }
      })
    },

    handSendCode () {
      const { mobile } = this.form
      if (this.captchaObj) {
        return this.captchaObj.verify()
      }

      axios({
        mehtod: 'GET',
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
      }).then(res => {
        const data = res.data.data
        window.initGeetest({
          // 以下配置参数来自服务端 SDK
          gt: data.gt,
          challenge: data.challenge,
          offline: !data.success,
          new_captcha: data.new_captcha,
          product: 'bind'// 隐藏式按钮
        }, (captchaObj) => {
          this.captchaObj = captchaObj
          // 这里可以调用验证实例 captchaObj 的实例方法
          captchaObj.onReady(function () {
            captchaObj.verify() // 显示验证码
          }).onSuccess(function () {
            const { geetest_challenge: challenge,
              geetest_challenge: seccode,
              geetest_validate: validate } =
              captchaObj.getValidate()
            axios({
              method: 'GET',
              url: `http://ttapi.research.itcast.cn/mp/v1_0/sms/codes/${mobile}`,
              params: {
                challenge,
                seccode,
                validate
              }
            }).then(res => {
              console.log(res.data)
            })
          })
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login-wrap{
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #ccc;
    .login-head{
      display: flex;
      justify-content: center;
      margin-bottom: 20px;
    }
    .login-from-wrap{
      background-color: #fff;
      padding: 60px;
      .btn-login{
        margin-top: 20px;
        width: 100%;
      }
    }
}
</style>
