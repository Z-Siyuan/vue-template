<template>
  <div class="login-container">
    <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form" auto-complete="on" label-position="left">
      <!-- 系统标题 -->
      <div class="title-container">
        <h3 class="title">后台管理系统</h3>
      </div>
      <!-- 用户名 -->
      <el-form-item prop="username">
        <span class="svg-container">
          <svg-icon icon-class="user" />
        </span>
        <el-input
          ref="username"
          v-model="loginForm.username"
          placeholder="用户名"
          name="username"
          type="text"
          tabindex="1"
          auto-complete="on"
        />
      </el-form-item>
      <!-- 密码 -->
      <el-form-item prop="password">
        <span class="svg-container">
          <svg-icon icon-class="password" />
        </span>
        <el-input
          :key="passwordType"
          ref="password"
          v-model="loginForm.password"
          :type="passwordType"
          placeholder="密码"
          name="password"
          tabindex="2"
          auto-complete="on"
          @keyup.enter.native="handleLogin"
        />
        <span class="show-pwd" @click="showPwd">
          <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
        </span>
      </el-form-item>

      <!-- 图形验证码 -->
      <el-form-item class="ImageCheck" prop="imageCode" style="border-radius:5px 0 0 5px;">
        <span class="svg-container">
          <svg-icon icon-class="tuxing" />
        </span>
        <el-input
          ref="imageCode"
          v-model="loginForm.imageCode"
          placeholder="验证码"
          name="imageCode"
          type="text"
          tabindex="3"
          auto-complete="on"
          @keyup.enter.native="handleLogin"
          @focus="focusVerifyCode"
          @input="changeVerifyCode"
        />
      </el-form-item>
      <a v-show="loginForm.showImageCode" href="javascript:;"><img :src="imgCode" @click="verifyCode"></a>

      <el-button :loading="loading" type="primary" style="width:100%;margin-bottom:30px;" @click.native.prevent="handleLogin" tabindex="4">登录</el-button>

    </el-form>
  </div>
</template>

<script>
import { Graphic } from '@/api/user'


export default {
  name: 'Login',
  data: function() {
    const validateUsername = (rule, value, callback) => {
      if (value.length < 3 || value.length > 8) {
        // this.$refs.username.focus()
        callback(new Error('请输入用户名3-8位'))
      } else {
        callback()
      }
    }
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        // this.$refs.password.focus()
        callback(new Error('密码不能少于6位'))
      } else {
        callback()
      }
    }
    const validateImageCode = (rule, value, callback) => {
      if (value.length < 4) {
        // this.$refs.imageCode.focus()
        callback(new Error('请输入图形验证码'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        username: '', // 用户名
        password: '', // 密码
        imageToken: '', // 凭据
        imageCode: '', // 图片验证码
        showImageCode: false // 是否显示图形验证码
      },
      loginRules: {
        username: [{ required: true, trigger: 'blur', validator: validateUsername }],
        password: [{ required: true, trigger: 'blur', validator: validatePassword }],
        imageCode: [{ required: true, trigger: 'blur', validator: validateImageCode }]
      },
      loading: false,
      passwordType: 'password',
      redirect: undefined,
      imgCode: '' // 图片地址
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        this.redirect = route.query && route.query.redirect
      },
      immediate: true
    }
  },
  methods: {
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    verifyCode() {
      Graphic().then(res => {
        this.loginForm.imageToken = res.verifytoken
        this.imgCode = `${res.qrcode}`
      })
    },
    focusVerifyCode() {
      if (!this.loginForm.showImageCode) {
        this.loginForm.showImageCode = true
        this.verifyCode()
      }
    },
    changeVerifyCode() {
      if (this.loginForm.imageCode.length === 4) {
        this.handleLogin()
      } else if (this.loginForm.imageCode.length > 4) {
        this.loginForm.imageCode = this.loginForm.imageCode.substr(0, 4)
      }
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
          this.$store.dispatch('user/login', this.loginForm).then(() => {
            this.$router.push({ path: this.redirect || '/' })
            this.loading = false
          }).catch(() => {
            this.loading = false
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    }
  }
}
</script>

<style lang="scss">
/* 修复input 背景不协调 和光标变色 */
/* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */

$bg:#283443;
$light_gray:#fff;
$cursor: #fff;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}

/* reset element-ui css */
.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;

    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $light_gray;
      height: 47px;
      caret-color: $cursor;

      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px $bg inset !important;
        -webkit-text-fill-color: $cursor !important;
      }
    }
  }

  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
}
</style>

<style lang="scss" scoped>
$bg:#2d3a4b;
$dark_gray:#889aa4;
$light_gray:#eee;

.login-container {
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  overflow: hidden;

  .login-form {
    position: relative;
    width: 520px;
    max-width: 100%;
    padding: 160px 35px 0;
    margin: 0 auto;
    overflow: hidden;
  }

  .ImageCheck {
    width:70%;
    float: left;
  }
  img {
    width:30%;
    height: 54px;
    line-height:50px;
    border-radius:0 5px 5px 0;
    border:1px solid #3e4957;
    border-left:none;
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 26px;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }
}
</style>
