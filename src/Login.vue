<template>
  <div class="login-container">
    <div class="login-inner">
      <div class="login-header">
        <div class="login-logo">
          <img src="./images/logo-round.png" alt="" width="150">
        </div>
        <div class="login-header-title">
          <a href="javascript:;" :class="loginMode? 'current': ''" @click="dealLoginMode(true)">验证码登录</a>
          <a href="javascript:;" :class="!loginMode? 'current': ''" @click="dealLoginMode(false)">密码登录</a>
        </div>
      </div>
    <div class="login-content">
       <form>
         <!-- 手机验证码登录部分 -->
         <div :class="loginMode ? 'current' : '' ">
           <section class="login-message">
             <input type="tel" placeholder="手机号" maxlength="11" v-model="phone">
             <button class="get-verification" v-if="!countDown" :class="{phone_right: phoneRight}" @click="getVerifyCode">获取验证码</button>
             <button class="get-verification" v-else disabled="disabled">
               已发送({{countDown}}s)
            </button>
           </section>
           <section class="login-verification">
             <input type="password" maxlength="8" placeholder="验证码">
             <!-- <input type="text" maxlength="18" placeholder="密码"> -->
           </section>
           <section class="login-hint">
            温馨提示：未注册帐号的用户账号，登录时将自动注册，且代表已同意
            <a href="javascript:;">服务协议与隐私政策</a>            
           </section>
         </div>
         <!-- 账号登录部分 -->
         <div :class="{current: !loginMode}">
           <section class="login-message">
             <input type="text" placeholder="账号" maxlength="11" v-model="user_name">
           </section>
           <section class="login-verification">
             <input type="password" maxlength="18" placeholder="密码" v-if="pwdMode" v-model="pwd">
             <input type="text" maxlength="18" placeholder="密码" v-else v-model="pwd">
             <div class="switch-show">
               <img @click="dealPwdMode(false)" :class="{on: pwdMode}" src="./images/hide_pwd.png" alt="" width="20">
               <img @click="dealPwdMode(true)" :class="{on: !pwdMode}" src="./images/show_pwd.png" alt="" width="20">
             </div>
           </section>
           <section class="login-message">
              <input type="text" maxlength="11" placeholder="验证码" v-model="captcha">
              <img ref="captcha" class="get-verification" src="http://localhost:3000/api/captcha" alt="captcha" @click.prevent="getCaptcha()">
           </section>
           <section class="login-hint">
            温馨提示：未注册帐号的用户账号，登录时将自动注册，且代表已同意
            <a href="javascript:;">服务协议与隐私政策</a>            
           </section>
         </div>
         <button class="login-submit" @click="login()">登录</button>
       </form>
         <button class="login-back" @click="$router.back()">返回</button>
    </div>
    </div>
  </div>
</template>
<script>
import {getPhoneCode, phoneCodeLogin, pwdLogin} from './../../api/index';
import { MessageBox } from 'element-ui';
import {mapActions} from 'vuex'
export default {
  data(){
    return {
      loginMode: true,
      user_name: '',//用户名
      pwd: '',//密码
      pwdMode: true,
      captcha: '',
      phone:'',
      countDown:0,//倒计时
    }
  },
  computed: {
   phoneRight(){
     return /^[1][3,4,5,6,7,8]{9}$/.test(this.phone)
   }
  },
  methods:{
  ...mapActions(['syncUserInfo']),
  dealLoginMode(flag){
    this.loginMode=flag
  },
  dealPwdMode(flag) { 
    this.pwdMode = flag;
      },
  getCaptcha() {
    this.$refs.captcha.src='http://localhost:3000/api/captcha?time=' + new Date();
  },
      async getVerifyCode() {
        // 2.1 开启倒计时
        if (this.phoneRight) {
          this.countDown = 60;
          this.intervalId = setInterval(() => {
            this.countDown--;
            // 判断
            if (this.countDown === 0) {
              clearInterval(this.intervalId);
            }
          }, 1000);
        }

        // 2.2 获取短信验证码
        let result = await getPhoneCode(this.phone);
        console.log(result);

        // 2.3 获取验证码失败
        if (result.err_code === 0) {
          // 提示信息
		      MessageBox({
              type: 'info',
              message: result.message,
			        showClose: true,
          });

          // 其他处理
          /*
          clearInterval(this.intervalId);
          this.countDown = 0;
          */
        }else if(!this.phone){
          	MessageBox({
            type: 'info',
            message: '手机号不能为空!!!',
			      showClose: true,
          })
        }else if(result.success_code === 200&&!this.phone){
			    MessageBox({
              type: 'info',
              message: `验证码为:${result.message}, 短信功能暂未开通,十分抱歉对您产生的不便`,
			        showClose: true,
          });
		    }
      },
      async login(){
        if (this.loginMode) {
          if(!this.phone) {
		        MessageBox({
              type: 'info',
              message: "请输入手机号码!",
			        showClose: true,
            });
            return;            
          }else if(!this.phoneRight) {
            	MessageBox({
              type: 'info',
              message: "请输入正确手机号码!",
			        showClose: true,
            });
            return;
          }
          if(!this.code) {
            MessageBox({
              type: 'info',
              message: '请输入验证码！',
              showClose: true,
            });
            return ; 
          }else if(!(/^\d{6}$/gi.test(this.code))) {
            MessageBox({
              type: 'info',
              message: "请输入正确的验证码!",
			        showClose: true,
            });
            return;
          }
          const result =await pwdLogin(this.user_name,this.pwd,this.captcha);
          if (result.success_code === 200) {
            this.userInfo = result.message;
			      this.phone = ''; // 手机号码
            this.countDown = 0; // 倒计时
			      clearInterval(this.intervalId);
            this.code = ''; // 验证码
			      this.pwd = ''; // 密码
            this.user_name = ''; // 账号
            this.captcha = '';  // 图形验证码
          }else {
            MessageBox({
              type: 'info' ,
              message: '登录失败，账号或验证码不正确!',
              showClose: true,
            })
          }
        }
        if(!this.userInfo.id) {
          MessageBox({
            type: 'info',
            message: this.userInfo.message,
            showClose: true
          });
        }else {
          this.syncUserInfo(this.userInfo);
          MessageBox({
            type: 'success',
            message: '登录成功!',
			      showClose: true,
          });   
          this.$router.back();
          window.localStorage.setItem("userInfo",JSON.stringify(this.userInfo));
          window.localStorage.removItem("adminInfo")
        }
      }
 }
  }
</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixins.styl"
 .login-container
    width 100%
    height 100%
    background #fff
    .login-inner
      padding-top 60px
      width 25%
      margin 0 auto
      .login-header
        .login-logo
          font-size 20px
          font-weight bold
          color mediumpurple
          text-align center
        .login-header-title
          paddig-top 40px
          padding-bottom 10px
          text-align center
          > a
            color #333
            font-size 14px
            padding-bottom 4px
            &:first-child
              margin-right 40px
            &.current
              color mediumpurple
              font-weight 700
              border-bottom 2px solid mediumpurple
      .login-content
        > form
          > div
            display none
            &.current
              display block
            input
              width 100%
              height 100%
              padding-left 8px
              box-sizing border-box
              border 1px solid #ddd
              border-radius 4px
              outline 0
              font 400 14px Arial
              &:focus
                border 1px solid mediumpurple
            .login-message
              position relative
              margin-top 16px
              height 48px
              font-size 14px
              background #fff
              .get-verification
                position absolute
                top 50%
                right 10px
                transform translateY(-50%)
                border 0
                outline none
                color #ccc
                font-size 14px
                background transparent     
                &.phone_right
                  color purple         
            .login-verification
              position relative
              margin-top 16px
              height 48px
              font-size 14px
              background #fff
              .switch-show
              .switch-show
                position absolute
                right 10px
                top 12px
                img
                  display none
                img.on
                  display block
            .login-hint
              margin-top 12px
              color #999
              font-size 12px
              line-height 20px
              > a
                color mediumpurple                             
          .login-submit
            display block
            width 100%
            height 42px
            margin-top 30px
            border-radius 4px
            background mediumpurple
            color #fff
            text-align center
            font-size 16px
            line-height 42px
            border 0
        .login-back
          display block
          width 100%
          height 42px
          margin-top 15px
          border-radius 4px
          background transparent
          border: 1px solid mediumpurple
          color mediumpurple
          text-align center
          font-size 16px
          line-height 42px
</style>