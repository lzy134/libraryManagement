<template>
  <div class="loginContainer">
    <div class="loginBox">
      <div class="loginWelcome">
        <span class="loginTitle">Welcome</span>
        <span class="loginSubTitle">图书管理系统</span>
      </div>
      <div class="loginMain">
        <el-form 
          ref="loginForm" 
          :model="loginForm" 
          :rules="loginRules"
          label-width="80px"
        >
          <el-form-item label="用户名：" prop="name">
            <el-input type="text" v-model="loginForm.name" v-popover:popover/>
          </el-form-item>
          <el-form-item label="密 码：" prop="password">
            <el-input type="password" v-model="loginForm.password" />
          </el-form-item>
          <el-form-item label="验证码：" class="verificationCode" prop="code">
            <el-row>
              <el-col :span="12">
                <el-input type="text" v-model="loginForm.code" />
              </el-col>
              <el-col :span="12">
                <div v-html="captchaUrl"  @click="updateCaptcha"></div>
              </el-col>
            </el-row>
            <!-- <span>2357</span> -->
          </el-form-item>
          <el-form-item >
            <!-- <el-checkbox v-model="loginRemember" @change="rememberPassword">记住密码</el-checkbox> -->
          </el-form-item>
          <el-form-item style="text-align: right">
            <el-button type="primary" @click="submitForm">登录</el-button>
            <el-button @click="resetForm">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
  import {getRequest, postRequest} from '../untils/api'
  export default {
    name: 'Login',
    data() {
      return {
        captchaUrl: '/api/captcha?time=' + new Date(),
        loginForm: {
          name: '',
          password: '',
          code: ''
        },
        loginRemember: false,
        loginRules: {
          name: [
            {required: true, message: '请输入用户名！', trigger: 'blur'}
          ],
          password: [
            {required: true, message: '请输入密码！', trigger: 'blur'}
          ],
          code: [
            {required: true, message: '请输入验证码！', trigger: 'blur'}
          ]
        }
      }
    },
    mounted() {
      this.updateCaptcha()
    },
    methods: {
      // 登录
      submitForm() {
        this.$refs.loginForm.validate((valid) => {
          if (valid) {
            postRequest('/login', this.loginForm).then(resp => {
              if(resp.code !== 0) {
                this.updateCaptcha()
                return this.$message.error(resp.msg)
              }
              this.$router.replace({name: 'index'})
              this.$message.success(resp.msg)
              sessionStorage.setItem('token', resp.token)
            })
          } else {
            return false;
          }
        });
      },
      // 重置
      resetForm() {
        this.$refs.loginForm.resetFields();
      },
      // 更新验证码
      updateCaptcha() {
        getRequest('/captcha').then(resp => {
          this.captchaUrl = resp
        })
      }
    }
  }
</script>

<style lang="less" scoped>
  .loginContainer {
    position: relative;
    width: 100%;
    height: 100%;
    // background-color: #409eff;
    background: url(../assets/imgs/bg.jpg) no-repeat 0 0 transparent;
    background-size: 100% 100%;
  }
  .loginBox {
    display: flex;
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -350px;
    margin-top: -200px;
    width: 700px;
    height: 400px;
    border-radius: 20px;

    box-shadow: 0 0 10px rgba(255,255,255,.7);
    .loginWelcome {
      width: 300px;
      height: 100%;
      background-color: rgba(255,255,255,.6);
      border-radius: 20px 0 0 20px;
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;

      .loginTitle {
        font-size: 26px;
        font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
        font-weight: 600;
      }
      .loginSubTitle {
        font-size: 30px;
        font-weight: 700;
        font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
        margin-top: 40px;
      }
    }
    .loginMain {
      width: 400px;
      height: 100%;
      background-color: #fff;
      border-radius: 0 20px 20px 0;
      padding: 20px;
      box-sizing: border-box;
      .el-form {
        margin-top: 50px;
        .verificationCode {
          .el-input {
            width: 120px;
            margin-right: 10px;
          }
          span {
            padding: 10px 20px;
            border: 1px solid #f1f1f1;
            border-radius: 4px;
          }
        }
      }
    }
  }
  .loginMain /deep/.el-popper[x-placement^=bottom] {
    width: 258px !important;
    left: 466px !important;
  }
</style>