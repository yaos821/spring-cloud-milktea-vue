<template>
  <div class="registry-container">
    <el-form ref="registryForm" :model="registryForm" :rules="registryRules" class="registry-form" auto-complete="on" label-position="left">

      <div class="title-container">
        <h3 class="title">恋上奶茶 后台管理系统</h3>
      </div>

      <el-form-item prop="role" tabindex="1">
        <span class="svg-container">
          <svg-icon icon-class="star" />
        </span>
        <el-select v-model="registryForm.type" placeholder="管理员类型">
          <el-option
            v-for="item in type"
            :key="item.value"
            :label="item.label"
            :value="item.value"
            :disabled="item.disabled"
          />
        </el-select>
      </el-form-item>

      <el-form-item prop="name">
        <span class="svg-container">
          <svg-icon icon-class="user" />
        </span>
        <el-input ref="name" v-model="registryForm.name" placeholder="管理员账户" name="name" type="text" tabindex="2" auto-complete="on" />
      </el-form-item>

      <el-form-item prop="password">
        <span class="svg-container">
          <svg-icon icon-class="password" />
        </span>
        <el-input :key="passwordType" ref="password" v-model="registryForm.password" :type="passwordType" placeholder="密码" name="password" tabindex="3" auto-complete="on" @keyup.enter.native="handleregistry" />
        <span class="show-pwd" @click="showPwd">
          <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
        </span>
      </el-form-item>

      <el-form-item prop="confim">
        <span class="svg-container">
          <svg-icon icon-class="password" />
        </span>
        <el-input :key="confimType" ref="confim" v-model="registryForm.confim" :type="confimType" placeholder="请再次确认密码" name="confim" tabindex="4" auto-complete="on" />
        <span class="show-pwd" @click="showConfim">
          <svg-icon :icon-class="confimType === 'password' ? 'eye' : 'eye-open'" />
        </span>
      </el-form-item>

      <el-form-item prop="email">
        <span class="svg-container">
          <i class="el-icon-mobile-phone" />
        </span>
        <el-input ref="email" v-model="registryForm.email" placeholder="电子邮箱" name="email" type="text" tabindex="5" auto-complete="on" />
      </el-form-item>

      <el-button :loading="loading" type="primary" style="width:100%;margin-bottom:30px;" @click.native.prevent="handleregistry">注册</el-button>

      <div class="center" style="text-align:center;">
        <router-link :to="{path:'login'}">
          <el-link type="info" :underline="false">
            已有账号？点击去登录！
          </el-link>
        </router-link>
      </div>
    </el-form>
  </div>
</template>

<script>

import { registry } from '@/api/admin'

export default {
  name: 'Registry',
  data() {
    const validateName = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('管理员账户不能为空'))
      } else {
        callback()
      }
    }
    const validatePassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('密码不能为空'))
      } else if (value.length < 6) {
        callback(new Error('密码不能少于6位'))
      } else {
        callback()
      }
    }
    const validateConfim = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.registryForm.password) {
        callback(new Error('两次输入密码不一致！'))
      } else {
        callback()
      }
    }
    const validEmail = (rule, value, callback) => {
      var pattern = /^[0-9a-zA-Z_.-]+[@][0-9a-zA-Z_.-]+([.][a-zA-Z]+){1,2}$/
      if (value === '') {
        callback(new Error('电子邮箱不能为空'))
      } else if (!pattern.test(value)) {
        callback(new Error('电子邮箱格式不正确'))
      } else {
        return callback()
      }
    }
    return {
      type: [{
        value: '0',
        label: '系统管理员',
        disabled: true
      }, {
        value: '1',
        label: '普通管理员'
      }, {
        value: '2',
        label: '观察员'
      }],
      registryForm: {
        name: '',
        password: '',
        confim: '',
        email: '',
        type: ''
      },
      registryRules: {
        name: [{ required: true, trigger: 'blur', validator: validateName }],
        password: [{ required: true, trigger: 'blur', validator: validatePassword }],
        confim: [{ required: true, trigger: 'blur', validator: validateConfim }],
        email: [{ required: true, trigger: 'blur', validator: validEmail }]
      },
      loading: false,
      passwordType: 'password',
      confimType: 'password'
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
    showConfim() {
      if (this.confimType === 'password') {
        this.confimType = ''
      } else {
        this.confimType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.confim.focus()
      })
    },
    handleregistry() {
      this.$refs.registryForm.validate(valid => {
        if (valid) {
          this.loading = true
          registry(this.registryForm).then(response => {
            if (response.code > 0) {
              this.$notify({
                title: '成功',
                message: '注册管理员账户成功',
                type: 'success',
                duration: 2000
              })
              // 注册后直接登录
              this.$store.dispatch('admin/login', {
                name: this.registryForm.name,
                password: this.registryForm.password
              }).then(() => {
                this.$router.push({ path: this.redirect || '/' })
                this.loading = false
              }).catch(() => {
                this.loading = false
              })
            } else {
              this.$notify({
                title: '失败',
                message: '注册管理员账户失败',
                type: 'danger',
                duration: -1
              })
            }
          })
        }
      })
    }
  }
}
</script>

<style lang="scss">
/* 修复input 背景不协调 和光标变色 */
/* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */

$bg: #283443;
$light_gray: #fff;
$cursor: #fff;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .registry-container .el-input input {
    color: $cursor;
  }
}

/* reset element-ui css */
.registry-container {
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
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;

.registry-container {
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  overflow: hidden;

  .registry-form {
    position: relative;
    width: 520px;
    max-width: 100%;
    padding: 160px 35px 0;
    margin: 0 auto;
    overflow: hidden;
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
