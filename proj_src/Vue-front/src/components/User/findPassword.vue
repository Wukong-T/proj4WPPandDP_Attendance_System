<template>
  <el-container style="height: 100%; background: antiquewhite">
    <el-container style="padding: 10px; line-height: 0; background: #66b5cd; margin: 0 auto">
      <el-row
        style="height: 100%; margin: 0 auto"
        type="flex"
        class="row-bg loginpageback"
        justify="space-around"
      >
        <el-row>
          <!-- 标题 -->
          <el-col :xs="0" :sm="12" :md="13" :lg="15" :xl="15">
            <div class="grid-content" style="margin-top: -10px">
              <h2
                style="font-size: 40px; z-index：2; position:absolute; left:0px; font-weight:700; color:#fff;"
              >
              </h2>
              <img
                src="../../assets/img/忘记密码.svg"
                style="background: #66b5cd"
                width="600px"
                alt=""
                class=""
              />
            </div>
          </el-col>
          <!-- 找回密码表单 -->
          <el-col :xs="24" :sm="9" :md="9" :lg="9" :xl="8">
            <div class="grid-content">
              <el-form
                :model="ruleForm"
                :rules="rules"
                ref="ruleForm"
                label-width="100px"
                class="myform"
              >
                <el-row>
                  <el-col :span="24" style="text-align: center;">
                    <div class="grid-content loginlogo">
                      <img
                        src="../../assets/img/找回密码.svg"
                        width="50%"
                        height="30%"
                        alt=""
                      />
                    </div>
                  </el-col>
                </el-row> 
                <el-form-item label="账号" prop="username">
                  <el-input
                    prefix-icon="el-icon-user-solid"
                    placeholder="请输入账号（工号）"
                    v-model.trim="ruleForm.username"
                  ></el-input>

                </el-form-item>
                <el-form-item label="邮箱 " prop="email">
                  <el-input
                    placeholder="请输入邮箱"
                    prefix-icon="el-icon-message"
                    v-model.trim="ruleForm.email"
                  ></el-input>
                </el-form-item>

                <el-form-item  label="选择用户" prop="choice" style="margin-left:5px; text-align: center" >
                  <el-select 
                  style="margin-right:120px; width:300px"
                    v-model="ruleForm.choice" 
                    placeholder="请选择用户类型"
                    prop="choice">
                    <el-option label="员工" value="员工"></el-option>
                    <el-option label="部门经理" value="部门经理"></el-option>
                    <el-option label="副经理" value="副经理"></el-option>
                    <el-option label="总经理" value="总经理"></el-option>
                    <!-- <el-option label="人事管理员" value="人事管理员" ></el-option> -->
                  </el-select>
                </el-form-item>

              <el-form-item label="邮箱验证码" prop="verifyCode">
                 <div >
                  <el-input 
                    placeholder="请输入验证码"
                    prefix-icon="el-icon-picture-outline"
                    v-model.trim="ruleForm.verifyCode"
                     style="width:150px"
                  ></el-input>

                  <el-button :type="color" 
                  style="margin-left:30px ; width:120px; text-align: center" 
                  plain 
                  v-loading.fullscreen.lock="checkLoading"
                  @click="getCheckemail()" 
                  :disabled="flag">{{content}}</el-button>
                </div>
              </el-form-item>

                <el-form-item label="输入新密码" prop="password" auto-complete="off">
                  <el-input
                    prefix-icon="el-icon-lock"
                    placeholder="请输入6位密码"
                    type="password"
                    v-model.trim="ruleForm.password"
                    show-password
                  ></el-input>
                </el-form-item>
                <el-form-item
                  label="确认密码"
                  prop="checkPass"
                  auto-complete="off"
                >
                  <el-input
                    placeholder="请再次输入密码"
                    prefix-icon="el-icon-lock"
                    type="password"
                    v-model.trim="ruleForm.checkPass"
                    show-password
                  ></el-input>
                </el-form-item>

                <el-form-item>
                  <div style="text-align: left">
                    <el-button
                      type="primary"
                      round
                      @click="submitForm('ruleForm')"
                      >确认修改</el-button
                    >
                    
                    <el-button round @click="resetForm('ruleForm')"
                      >重置</el-button>
                    <el-button type="info" round plain @click="back('ruleForm')"
                      >返回</el-button>
                  </div>
                </el-form-item>
              </el-form>
            </div>
          </el-col>
 
        </el-row>
      </el-row>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    var validatePass = (rule, value, callback) => {
      let reg = new RegExp("[ ]");
      if (value === "") {
        callback(new Error("请输入密码"));
      } else if (/[\u4E00-\u9FA5]/g.test(value)) {
        callback(new Error("不能输入中文!"));
      } else if (reg.test(value)) {
        callback(new Error("不能输入空格字符"));
      } else {
        if (this.ruleForm.checkPass !== "") {
          this.$refs.ruleForm.validateField("checkPass");
        }
        callback();
      }
    };
    var validatePass2 = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.ruleForm.password) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    };
  

    return {
       content: '获取验证码', // 获取验证码按钮文字
       flag: false,//按钮是否可取
       totalTime: 30,//倒计时时间
       color:"success",//按钮颜色类型
       checkLoading:false,//缓存
       
       codeTime:30,

      ruleForm: {
        username: "",
        password: "",
        email: "",
        checkPass: "",
        verifyCode:"",
        choice:[],
      },
       
      successResponse: [],

      rules: {
        verifyCode: [
          { required: true, message: "请输入验证码", trigger: "blur" },
          ],
        username: [
          { required: true, message: "请输入账号", trigger: "blur" },
          { min: 10, message: "请输入正确的职工号", trigger: "blur" },
          { max: 10, message: "请输入正确的职工号", trigger: "blur" },
          {
            validator: function (rule, value, callback) {
              //校验空格
              let reg = new RegExp("[ ]");
              if (reg.test(value)) {
                callback(new Error("不能输入空格字符"));
              } else {
                //校验通过
                callback();
              }
            },
            
          },
        ],
        choice: [{ required: true, message: "请选择用户", trigger: "blur" }],

        email: [
          { required: true, message: "请输入邮箱地址", trigger: "blur" },
          {
            type: "email",
            message: "请输入正确的邮箱地址",
            trigger: ["blur", "change"],
          },
        ],
        password: [
          { required: true, trigger: "blur", validator: validatePass },
          { max: 12, message: "长度不能超过12个字符", trigger: "blur" },
          { min: 6, message: "长度不能少于6个字符", trigger: "blur" },
        ],
        checkPass: [
          { required: true, trigger: "blur", validator: validatePass2 },
        ],
      },
    };
  },
  methods: {

    submitForm(formName) {
          this.$refs[formName].validate((valid) => {
        if (valid) {       
          this.$axios
          .post('http://localhost:8181/user/updatePassword/', {
            username:this.ruleForm.username,
            password:this.ruleForm.password,
            choice:this.ruleForm.verifyCode,//验证码
          }
          ).then(successResponse => {

            switch(successResponse.data){
              case "success":
                this.$router.push({ name: "login" });
                window.sessionStorage.setItem("loginuser",this.ruleForm.username);
                window.sessionStorage.setItem("loginchoice",this.ruleForm.choice);
                this.$message({message: "修改成功，请重新登录",type: "success", }); 
                break;
              case "error":
                this.$message.error('验证码错误！');
                break;
              case "empty":
                this.$message.error('账号不存在！');
                break;
            }

          });
        } 
        else {
          this.$message.error('修改失败！');
        }
      
      });
    },

    getCheckemail(){
      //发送邮箱
        this.checkLoading = true;
      this.$axios.post('http://localhost:8181/user/getCheckemail', {
        username: this.ruleForm.username,
        email: this.ruleForm.email,
        choice: this.ruleForm.choice
        }).then(successResponse => {
        
        switch(successResponse.data){
          case "addressError":
            this.checkLoading = false;
            this.$message({message: "请确实绑定邮箱!",type: "warning",}); 
            break;
            
            case "error":
              this.checkLoading = false;
              this.$message({message: "邮箱地址错误或邮箱不存在!",type: "warning",}); 
            break;    

            case "userError":
              this.checkLoading = false;
              this.$message({message: "用户或选择用户错误!",type: "warning",});
            break;   

            case "empty":
              this.checkLoading = false;
              this.$message({message: "用户不存在",type: "error",}); 
            break;

            case "success":
              this.checkLoading=false;
              this.$message({message: "获取成功!请查看邮箱！",type: "success",}); 

                //验证码发送成功后按钮变化
                  this.flag = true;//点击之后设置按钮不可取
                  this.content = this.totalTime + "s后重新发送";//按钮内文本
                  this.color  = "info"//按钮颜色变化
                //时间控制
                let clock = window.setInterval(() => {
                  this.totalTime--;
                  this.content = this.totalTime + "s后重新发送";
                  if (this.totalTime < 0) {
                    window.clearInterval(clock);
                    this.content = "重新发送验证码";
                    this.totalTime = 30;
                    this.flag = false; //这里重新开启
                    this.color = "success";
                  } 
                }, 1000);
            break;
        }

        });
        
    },

    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    back(formName){
      this.$refs[formName].resetFields();
      sessionStorage.clear();
      this.$router.push({ name: "login" });
    }

  },
};
</script>

<style>
.register {
  background: url("../../assets/img/实名认证.svg");
  background-size: 125% 130%;
  /* border: 1px; */
  width: 100%;
}

.myform {
  padding: 20px;
  z-index: 2;
  width: 400px;
  background: #fff;
  -webkit-border-radius: 4px;
  border-radius: 4px;
  background-image: -webkit-gradient(
    linear,
    left top,
    left bottom,
    from(#fff),
    to(#f4f4f4)
  );
  background-image: -webkit-linear-gradient(top, #fff, #f4f4f4);
  background-image: -moz-linear-gradient(top, #fff 0, #f4f4f4 100%);
  background-image: linear-gradient(-180deg, #fff, #f4f4f4);
  -webkit-box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.1);
  box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.1);
}
</style>
