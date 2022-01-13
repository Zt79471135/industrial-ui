<template>
   <div class="login">
      <div class="login_box">
         <!-- 顶部标题 -->
         <div class="login_box_title">
            <img src="../assets/images/login/logo.png"
                 class="login_box_icon" />
            <div class="title_text">
               <div class="title_text_1">中山工业互联网平台</div>
               <div class="title_text_2">Zhongshan Industrual Internet Platform</div>
            </div>
         </div>
         <!-- 登录窗 -->
         <el-form ref="loginForm"
                  :model="loginForm"
                  :rules="loginRules"
                  class="login-form">
            <!-- 边框img -->
            <img src="../assets/images/login/login_top.png"
                 class="login_border_img1" />
            <img src="../assets/images/login/login_btom.png"
                 class="login_border_img2" />
            <!-- 边框img -->
            <div class="title">
               <div class="line_icon"></div>
               <div class="title_login">用户登录 USER LOGIN</div>
            </div>
            <el-form-item prop="username">
               <el-input v-model="loginForm.username"
                         type="text"
                         auto-complete="off"
                         placeholder="账号">
                  <svg-icon slot="prefix"
                            icon-class="user"
                            class="el-input__icon input-icon" />
               </el-input>
            </el-form-item>
            <el-form-item prop="password">
               <el-input v-model="loginForm.password"
                         type="password"
                         auto-complete="off"
                         placeholder="密码"
                         @keyup.enter.native="handleLogin">
                  <svg-icon slot="prefix"
                            icon-class="password"
                            class="el-input__icon input-icon" />
               </el-input>
            </el-form-item>
            <!-- 验证码1 -->
            <el-form-item prop="code"
                          v-if="captchaOnOff">
               <el-input v-model="loginForm.code"
                         auto-complete="off"
                         placeholder="验证码"
                         style="width: 63%"
                         @keyup.enter.native="handleLogin">
                  <svg-icon slot="prefix"
                            icon-class="validCode"
                            class="el-input__icon input-icon" />
               </el-input>
               <div class="login-code">
                  <img :src="codeUrl"
                       @click="getCode"
                       class="login-code-img" />
               </div>
            </el-form-item>

            <!-- 验证码2 -->
            <div style="width: 100%; margin: 15px 0">
               <vue-simple-verify borderColor="rgba(3, 180, 245, 0.1)"
                                  bgColor="transparent"
                                  :barBackground="barBackground"
                                  ref="verify"
                                  movedColor="#03b4f5"
                                  tips="按住滑块向右滑动进行验证"
                                  @success="handelSlidingEvent" />
            </div>

            <div class="Ispassword">
               <el-checkbox v-model="loginForm.rememberMe"
                            style="color: #ccc">记住密码</el-checkbox>

               <el-tooltip class="item"
                           effect="dark"
                           placement="top-start">
                    <div slot="content">
                        　　<span>请联系管理员重置密码</span> <br />
                        　  <span>电话：400-888-8888</span>
                    </div>
                  <span class="password2">忘记密码？</span>
               </el-tooltip>
            </div>

            <el-form-item style="width: 100%; margin-top: 30px; margin-bottom: 50px">
               <el-button :loading="loading"
                          size="medium"
                          type="primary"
                          style="width: 100%"
                          @click.native.prevent="handleLogin">
                  <span v-if="!loading">登 录</span>
                  <span v-else>登 录 中...</span>
               </el-button>
               <div style="float: right"
                    v-if="register">
                  <router-link class="link-type"
                               :to="'/register'">立即注册</router-link>
               </div>
            </el-form-item>
         </el-form>
      </div>

      <!--  底部  -->
      <div class="el-login-footer">
         <span>Copyright © 2018-2021 ruoyi.vip All Rights Reserved.</span>
      </div>
   </div>
</template>

<script>
import VueSimpleVerify from 'vue-simple-verify'
import { getCodeImg } from '@/api/login'
import Cookies from 'js-cookie'
import { encrypt, decrypt } from '@/utils/jsencrypt'
import a from '@/assets/images/login/moveBtn.png';

export default {
   name: 'Login',
   components: {
      VueSimpleVerify,
   },
   data() {
      return {
         codeUrl: '',
          barBackground: `url(${a}) 100% 100% no-repeat;`,
         loginForm: {
            username: 'admin',
            password: 'admin123',
            rememberMe: false,
            code: '',
            uuid: '',
         },
         loginRules: {
            username: [
               {
                  required: true,
                  trigger: 'blur',
                  message: '请输入您的账号',
               },
            ],
            password: [
               {
                  required: true,
                  trigger: 'blur',
                  message: '请输入您的密码',
               },
            ],
            code: [
               {
                  required: true,
                  trigger: 'change',
                  message: '请输入验证码',
               },
            ],
         },
         loading: false,
         // 验证码开关
         captchaOnOff: true,
         // 注册开关
         register: false,
         // 滑动验证
         SlidingCheck: false,
         redirect: undefined,
      }
   },
   watch: {
      $route: {
         handler: function (route) {
            this.redirect = route.query && route.query.redirect
         },
         immediate: true,
      },
   },
   created() {
      this.getCode()
      this.getCookie()
   },
   methods: {
      getCode() {
         getCodeImg().then((res) => {
            console.log(res, '登录数据')
            this.captchaOnOff =
               res.captchaOnOff === undefined ? true : res.captchaOnOff
            if (this.captchaOnOff) {
               this.codeUrl = 'data:image/gif;base64,' + res.img
               this.loginForm.uuid = res.uuid
            }
         })
      },
      getCookie() {
         const username = Cookies.get('username')
         const password = Cookies.get('password')
         const rememberMe = Cookies.get('rememberMe')
         this.loginForm = {
            username:
               username === undefined ? this.loginForm.username : username,
            password:
               password === undefined
                  ? this.loginForm.password
                  : decrypt(password),
            rememberMe: rememberMe === undefined ? false : Boolean(rememberMe),
         }
      },
      // 滑动成功
      handelSlidingEvent() {
         this.SlidingCheck = true
      },
      handleLogin() {
         if (this.SlidingCheck !== true) {
            this.$message({
               message: '请进行滑动验证！',
               type: 'warning',
            })
            return
         }
         this.$refs.loginForm.validate((valid) => {
            if (valid) {
               this.loading = true
               if (this.loginForm.rememberMe) {
                  Cookies.set('username', this.loginForm.username, {
                     expires: 30,
                  })
                  Cookies.set('password', encrypt(this.loginForm.password), {
                     expires: 30,
                  })
                  Cookies.set('rememberMe', this.loginForm.rememberMe, {
                     expires: 30,
                  })
               } else {
                  Cookies.remove('username')
                  Cookies.remove('password')
                  Cookies.remove('rememberMe')
               }
               this.$store
                  .dispatch('Login', this.loginForm)
                  .then(() => {
                     this.$router
                        .push({ path: this.redirect || '/' })
                        .catch(() => {})
                     this.SlidingCheck = false
                  })
                  .catch(() => {
                     this.loading = false
                     this.SlidingCheck = false
                     this.$refs.verify.reset()
                     if (this.captchaOnOff) {
                        this.getCode()
                     }
                  })
            }
         })
      },
   },
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.login {
   display: flex;
   justify-content: center;
   align-items: center;
   height: 100%;
   background-image: url('../assets/images/login/login_bgI.jpg');
   background-size: cover;
}

.title {
   display: flex;
   align-items: center;
   width: 100%;
   height: 40px;
   margin-bottom: 20px;
   .line_icon {
      width: 4px;
      height: 20px;
      background: #03b4f5;
      margin-right: 6px;
      display: block;
   }
   .title_login {
      color: #03b4f5;
      font-size: 18px;
   }
}
.login_box {
   display: flex;
   align-items: center;
   justify-content: center;
   flex-direction: column;
   width: 400px;
   .login_box_title {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100%;
      margin-bottom: 40px;
      .login_box_icon {
         width: 50px;
         height: 50px;
      }
      .title_text {
         display: flex;
         flex-direction: column;
         margin-left: 15px;
         .title_text_1 {
            font-size: 30px;
            color: #03c2ff;
            font-weight: 500;
         }
         .title_text_2 {
            font-size: 12px;
            color: #016fa7;
            font-weight: 500;
            text-indent: 1em;
         }
      }
   }
}
.login_border_img1 {
   position: absolute;
   top: 0;
   left: 0;
   right: 0;
   width: 100%;
   height: 15px;
}
.login_border_img2 {
   position: absolute;
   bottom: 0;
   left: 0;
   right: 0;
   width: 100%;
   height: 15px;
}
.login-form {
   border-radius: 6px;
   background: rgba(3, 180, 245, 0.2);
   width: 100%;
   padding: 25px 25px 5px 25px;
   position: relative;
   .el-input {
      height: 38px;
      input {
         height: 38px;
      }
   }
   .input-icon {
      height: 39px;
      width: 14px;
      margin-left: 2px;
      color: #03b4f5;
   }
}
.login-tip {
   font-size: 13px;
   text-align: center;
   color: #bfbfbf;
}
.login-code {
   width: 33%;
   height: 38px;
   float: right;
   img {
      cursor: pointer;
      vertical-align: middle;
   }
}
.el-login-footer {
   height: 40px;
   line-height: 40px;
   position: fixed;
   bottom: 0;
   width: 100%;
   text-align: center;
   color: #fff;
   font-family: Arial;
   font-size: 12px;
   letter-spacing: 1px;
}
.login-code-img {
   height: 38px;
}
.Ispassword {
   display: flex;
   justify-content: space-between;
   align-items: center;
}
.password2 {
   font-size: 14px;
   color: #ccc;
   cursor: pointer;
   &:hover {
      color: #03b4f5;
   }
}

// 更改elment表单背景色
::v-deep .el-input__inner {
   color: #fff;
   background-color: transparent !important;
   border: 1px solid rgba(3, 180, 245, 0.1) !important;
}
::v-deep .el-button--primary {
   background-color: #03b4f5;
   border-color: #03b4f5;
}
::v-deep .el-button--primary:hover,
.el-button--primary:focus {
   background-color: #03b4f5;
   border-color: #03b4f5;
}
::v-deep .el-checkbox__input.is-checked + .el-checkbox__label {
   color: #03b4f5 !important ;
}

// 修改滑动组件效果
::v-deep .simple-verify .verify-bar {
    display: flex;
    align-items: center;
    justify-content: center;
   overflow: hidden !important;
   .icon {
       width: 44px;
       height: 30px;
    //   animation: movelve 0.8s infinite linear !important;
       background-size:100% 100%;
       background-repeat:no-repeat;
   }
}
::v-deep .simple-verify .verify-bar:hover{
   
   .icon {
      animation: none !important;
   }
}
//动画
@keyframes movelve {
   from {
      transform: translateX(0);
   }

   to {
      transform: translateX(60px);
   }
}
</style>
