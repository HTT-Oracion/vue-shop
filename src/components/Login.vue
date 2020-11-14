<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="" />
      </div>
      <!-- 登录表单区域 -->
      <!-- ref 表单实例对象 :model 绑定表单数据 :rules 表单规则 -->
      <el-form
        ref="loginFormRef"
        :model="loginForm"
        :rules="loginFormRUles"
        class="login_form"
      >
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input
            v-model="loginForm.username"
            prefix-icon="iconfont icon-user"
          ></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            prefix-icon="iconfont icon-3702mima"
            type="password"
          ></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
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
      loginFormRUles: {
        // 验证用户名是否合法
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '长度在 3 到 10 个字符',
            trigger: 'blur'
          }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 4,
            max: 16,
            message: '长度在 4 到 16 个字符',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    resetLoginForm () {
      // resetFields是 elementUI的一个函数 用于重置
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      // 登录  预验证
      this.$refs.loginFormRef.validate(async (valid) => {
        if (!valid) return
        const { data } = await this.$http.post('login', this.loginForm)
        if (data.meta.status === 200) {
          // 登录成功之后的token 保存到客户端的 sessionStorage中
          //   项目中除了登录之外的其他API接口，必须在登录之后才能访问
          //   token值应在当前网站打开期间生效，所以将token保存在 sessionStorage中
          // 通过编程式导航跳转到后台主页 /home
          window.sessionStorage.setItem('token', data.data.token)
          this.$router.push('/home')
          return this.$message.success('登录成功!')
        } else {
          return this.$message.error('登录失败，请重试！')
        }
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
.login_box {
  position: absolute;
  width: 450px;
  height: 300px;
  background: #fff;
  border-radius: 3px;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  .avatar_box {
    position: absolute;
    top: 0;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 130px;
    height: 130px;
    border: 1px solid #eee;
    background-color: #fff;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}
.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
