<template>
  <div class="register-content">
    <form action="#" @submit.prevent="">
      <input type="text" v-model.trim.lazy="userName" placeholder="用户名" @blur="authUserNameFun" />
      <span>{{userNameText}}</span>
      <input type="text" v-model.trim.lazy="email" placeholder="邮箱" @blur="authEmailFun" />
      <span>{{emailText}}</span>
      <input type="password" v-model.trim.lazy="passWord" placeholder="密码" @blur="authPassWordFun" />
      <span>{{passWordText}}</span>

      <div class="authCode">
        <input type="text" v-model.trim.lazy="authCode" placeholder="验证码" @blur="authAuthCodeFun">
        <div v-html="codeImg" @click="initAuthCode" class="img"></div>
      </div>
      <span>{{codeText}}</span>

      <div class="emailCode">
        <input type="text" v-model.trim="emailCode" placeholder="邮箱验证码">
        <button class="img" @click="getEmailCode" :disabled="disabledCodeSend">{{ EmailCodeText }}</button>
      </div>

      <button @click="register">注册</button>

      <div class="router-link">
        <router-link to="/login">已有账号，去登录</router-link>
        <router-link to=""></router-link>
      </div>
    </form>
  </div>
</template>

<script>
import { authCodeApi } from '@/apis/authCode.js'
export default {
  data() {
    return {
      userName: '',
      email: '',
      passWord: '',
      authCode: '',
      emailCode: '',
      codeImg: '',
      // 默认值可以为 true  进入界面是不会显示输入错误提示
      // 提交表单时 调用一次验证函数即可
      authUserName: true,
      authEmail: true,
      authPassWord: true,
      authAuthCode: true,
      EmailCodeText: '获取验证码',
      userNameText: '',
      emailText: '',
      codeText: '',
      passWordText: '',
      disabledCodeSend: false
    }
  },
  created() {
    this.initAuthCode()
  },
  methods: {
    async initAuthCode() {
      // 刷新时清空输入框
      this.authCode = ''
      const { data: res } = await authCodeApi()
      this.codeImg = res
    },
    // 用户第一次进入界面 不需要验证  输入后再验证
    // 不知道咋写方便 就这样 能用就行(doge)
    async authUserNameFun() {
      if (!/^[a-z0-9_-]{3,16}$/.test(this.userName)) {
        this.userNameText = '用户名有误'
        return false
      }
      const { data: res } = await this.$http.post('/api/user/register/find', { userName: this.userName })
      if (res.status === 'no') {
        this.userNameText = '该用户名已被占用'
        return false
      }
      this.userNameText = ''
      return true
    },
    async authEmailFun() {
      if (!/\w[-\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\.)+[A-Za-z]{2,14}/.test(this.email)) {
        this.emailText = '邮箱格式有误'
        return false
      }
      const { data: res } = await this.$http.post('/api/user/register/find', { email: this.email })
      if (res.status === 'no') {
        this.emailText = '该邮箱已注册'
        return false
      }
      this.emailText = ''
      return true
    },
    authPassWordFun() {
      if (!/^[a-z0-9_-]{6,18}$/.test(this.passWord)) {
        this.passWordText = '密码格式有误'
        return false
      }
      this.passWordText = ''
      return true
    },
    authAuthCodeFun() {
      if (!/^[a-zA-Z0-9]{4,4}$/.test(this.authCode)) {
        this.codeText = '验证码有误'
        return false
      }
      this.codeText = ''
      return true
    },

    async getEmailCode() {
      if (this.authUserNameFun() && this.authPassWordFun() && this.authAuthCodeFun()) {
      } else {
        return this.$message({
          type: 'error',
          message: '表单填写有误'
        })
      }

      // 1. 发起请求 发送验证码
      const { data: res } = await this.$http.post('/api/user/register/auth', {
        //  userName, email, passWord, authCode
        userName: this.userName,
        email: this.email,
        passWord: this.passWord,
        authCode: this.authCode
      })

      if (res.code === 200) {
        this.$message({
          type: 'success',
          message: '发送成功!'
        })
      } else {
        this.$message({
          type: 'error',
          message: '发送失败'
        })
      }
      // 2. 改为倒计时
      this.disabledCodeSend = true
      let time = 60
      const codeTimeOut = setInterval(() => {
        if (time === 0) {
          this.disabledCodeSend = false
          this.EmailCodeText = '重新获取'
          clearInterval(codeTimeOut)
          return
        }
        this.EmailCodeText = time
        time--
      }, 1000)
    },
    register() {
      this.authUserNameFun()
      this.authPassWordFun()
      this.authAuthCodeFun()
      if (this.authUserNameFun() && this.authPassWordFun() && this.authAuthCodeFun()) {
      } else {
        return this.$message({
          type: 'error',
          message: '表单填写有误'
        })
      }
      this.$http({
        // 两个 register 不是 bug
        url: '/api/user/register/register',
        method: 'POST',
        data: {
          userName: this.userName,
          email: this.email,
          passWord: this.passWord,
          authCode: this.authCode,
          emailCode: this.emailCode
        },
        // headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        // withCredentials: true
        withCrendentials: true
        // SameSite: true
      }).then(res => {
        console.log(res)
        if (res.data.code === 200) {
          this.$alert('注册成功！', '好耶！', {
            confirmButtonText: '返回首页',
            callback: action => {
              // this.$router.replace('')
              // window.location.href = '/'
              window.location.href = '/login'
            }
          })
          return
        }
        this.$alert(`注册失败！ ${res.data.msg}`, '坏耶！', {
          confirmButtonText: '确定',
          callback: action => {
            // 刷新验证码
            this.initAuthCode()
          }
        })
      })
      // if (res.code === 200) {
      // }
    }
  }
}
</script>

<style lang="less" scoped>
.register-content {
  margin-top: 70px;
  width: 100%;
  display: flex;
  justify-content: center;
  form {
    margin: 50px 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 40%;
    min-width: 300px;
    max-width: 400px;
    height: 400px;
    background-color: #fff;
    padding: 20px;
    box-sizing: border-box;
    // align-items: s;
    justify-content: space-between;
    span {
      font-size: 10px;
      width: 100%;
      text-align: left;
      margin-top: -1.3em;
      color: rgb(255, 81, 81);
    }
    input,
    button {
      height: 40px;
      font-size: 18px;
      &::placeholder {
        font-size: 16px;
      }
    }
    input {
      background-color: rgb(241, 239, 241);
      border: none;
      padding-left: 0.7em;
      width: 100%;
    }
    .authCode,
    .emailCode {
      display: flex;
      justify-content: space-between;
      width: 100%;
      input {
        width: 50%;
      }
      .img {
        width: 40%;
        background-color: rgb(238, 238, 238);
        display: flex;
        align-items: center;
        justify-content: center;
        svg {
          width: 100%;
          height: 100px;
        }
      }
    }
    .emailCode {
      .img {
        background-color: rgb(157, 224, 255);
        border-radius: 5px;
      }
    }
    button {
      background-color: #fff;
      border: none;
      width: 50%;
      background-color: rgb(51, 156, 255);
      border-radius: 50px;
      color: #fff;
    }
    .router-link {
      width: 100%;
      display: flex;
      justify-content: space-around;
      a {
        color: rgb(126, 126, 126);
        &:hover {
          color: #aaa;
        }
      }
    }
  }
}
</style>
