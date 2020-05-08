<template>
  <div class="login">
     <LoginHeader> 
        <el-form slot="container" :model="ruleForm" :rules="rules" ref="ruleForm"   label-position="left" label-width="0px" >
          <div class="title">
            <h3>账号密码登录</h3>
          </div>
          <!-- username-->
        <el-form-item prop="username" >
            <el-input type="pwd" v-model="ruleForm.username" autocomplete="off" placeholder="账号">
               <i slot="prefix" class="el-icon-user"></i>
            </el-input>
            
          </el-form-item>
      
          <el-form-item prop="pwd" >
            <el-input type="password" v-model="ruleForm.pwd" autocomplete="off" placeholder="密码" >
               <i slot="prefix" class="el-icon-lock"></i>
               </el-input>
          
           
          </el-form-item>

   <!-- 登录button -->
            <el-form-item >
            <el-button type="primary" style="width:100%;" :loading="isLogin"   @click.native.prevent="handleSubmit">登录</el-button>
          </el-form-item>

    <!-- 7天登录和忘记密码 -->
          <el-form-item >
            <el-checkbox v-model="ruleForm.autoLogin" :checked="ruleForm.autoLogin">7天内自动登录</el-checkbox>
              <el-button  @click="$router.push('/password')"   type="text" class="forget">忘记密码</el-button>
          </el-form-item>
        </el-form>

     

     </LoginHeader>
  </div>
</template>

<script lang="ts">
import {Component, Vue, Provide  } from 'vue-property-decorator' // 对类和函数进行扩充
import {State, Getter, Mutation, Action} from "vuex-class";
import LoginHeader from './LoginHeader.vue';
@Component({
  components:{
    LoginHeader
    }
})
export default class Login extends Vue {
  //  存储用户信息
  @Action("setUser") setUser:any;
  @Provide() isLogin:boolean = false;
 @Provide() ruleForm: {
    username: String;
    pwd: String;
    autoLogin: boolean;
  } = {
    username: "",
    pwd: "",
    autoLogin: true // 是否自动登录
  };

  @Provide() rules = {
    username: [{ required: true, message: "请输入账号", trigger: "blur" }],
    pwd: [{ required: true, message: "请输入密码", trigger: "blur" }]
  };

  handleSubmit():void{
    (this.$refs['ruleForm'] as any).validate((valid:boolean)=>{
      if(valid){
       this.isLogin = true;
        (this as any).$axios.post("/api/users/login",this.ruleForm).then((res:any)=>{
            this.isLogin = false;
            //  console.log(res.data);
            // 存储token
            localStorage.setItem('tsToken', res.data.token);
             // 存储 vuex
             this.setUser(res.data.token);
            // 登录成功跳转
              (this as any).$router.push('/')
        }).catch(()=>{
            this.isLogin = false;
        })
      }
    })
  }
}
</script>
<style lang="scss" scoped>
.login{
  width: 100%;
  height: 100%;
}
.title{
  margin: 0px auto 40px auto;
  text-align: center;
  color: #505458;
}
i {
  font-size: 14px;
  margin-left: 8px;
}
.forget{
  float: right;
}

</style>
