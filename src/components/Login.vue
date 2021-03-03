<template>
  <div class="login_container">
    <!-- 登录框绘制 -->
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/login_log.png" alt="">
      </div>
      <!-- 登录表单 -->
      <el-form :model="loginForm" :rules="loginFormRules" ref="loginFormRef" label-width="0px" class='login_form'>
        <!-- 用户名 -->
        <el-form-item prop='username'>
          <el-input v-model='loginForm.username' prefix-icon="icon-users"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop='password'>
          <el-input v-model='loginForm.password' prefix-icon="icon-key" type="password"></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class='btns'>
            <el-button type="primary" @click="login">登录</el-button>
            <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 登录表单的数据绑定对象
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 表单验证规则对象
      loginFormRules: {
        // 验证用户名是否合法
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 5, max: 15, message: '长度在 5 到 15 个字符', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入用户密码', trigger: 'blur' },
          { min: 6, max: 10, message: '长度在 6 到 10 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 点击重置按钮，组件重置
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    // 点击登录按钮，进行登录
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return false
        // 将具体相应对象data重命名为res
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登录失败')
        this.$message.success('登录成功')
        // 登录成功，将res中的token保存在sessioonStorage中
        window.sessionStorage.setItem('token', res.data.token)
        // 页面跳转
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box{
  position: absolute;
  top:50%;
  left:50%;
  transform:translate(20%,-50%);
  width:450px;
  height: 300px;
  background-color: #fff;
  border-radius: 10px;

  .avatar_box{
    position: absolute;
    left:50%;
    //将盒子移动到登录盒子的50%位置
    height: 75px;
    width:75px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding:8px;
    box-shadow:0 0 10px #ddd;
    transform: translate(-50%,-50%);//回撤自身宽度的50%
    background-color: #fff;
    img {
      height: 100%;
      width: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}
.login_form{
  position: absolute;
  bottom: 0;
  width: 100%;
  padding:0 20px;
  box-sizing: border-box;
}
.btns{
  //居右对齐
  display:flex;
  justify-content: flex-end;
}
</style>
