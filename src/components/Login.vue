<template>
  <div class="login">
    <h1>登陆</h1>
    <el-container>
      <el-main>
        <el-row>
          <el-col :xs="24" :sm="16" :md="12" :lg="8" :xl="4" >
            <div class="item">
              <el-select v-model="prefix_num" placeholder="请选择">
                <el-option
                  v-for="item in prefix"
                  :key="item.id"
                  :label="item.label"
                  :value="item.value">
                </el-option>
              </el-select>
              <el-input v-model="phonenum" type="phone" label="手机号码" autofocus="true" @blur.prevent="check"></el-input>
            </div>
            <div class="item">
              <el-input v-model="code" type="text" placeholder="请输入验证码"></el-input>
              <span>
                <el-button v-show="show" @click="getCode" type="default">获取验证码</el-button>
                <el-button v-show="!show" disabled type="info">重新获取{{second}}s</el-button>
              </span>
            </div>
            <div class="item">
              <el-input v-model="password" type="password" placeholder="密码(不少于6位)"></el-input>
            </div>
            <el-button @click="submit" type="primary">登陆</el-button>
          </el-col>
        </el-row>
      </el-main>
    </el-container>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      phonenum: '', // 手机号
      code: '', // 验证码
      prefix_num: '+86',
      prefix: [
        {id: 1, label: '中国大陆', value: '+86'},
        {id: 2, label: '中国香港', value: '+852'},
        {id: 3, label: '中国澳门', value: '+853'},
        {id: 4, label: '中国台湾', value: '+886'},
        {id: 5, label: '日本', value: '+81'},
        {id: 6, label: '美国', value: '+1'}
      ],
      password: '', // 密码
      second: '', // 倒计时
      flag: false, // 防止手机号不正确时点击获取验证码
      show: true, // button显示
      timer: null
    }
  },
  methods: {
    getCode () {
      const TIME_COUNT = 60
      if (this.phonenum === '') {
        this.$message('必须输入手机号码！')
        return false
      } else if (this.flag === false) {
        this.$message('必须输入正确的手机号码！')
        return false
      } else {
        if (!this.timer) {
          this.show = false
          this.second = TIME_COUNT
          this.$ajax.get('https://easy-mock.com/mock/5b2385e3debe3c5977248a16/wscn/captcha')
            .then(r => {
              // console.log(r.data.data.captcha)
              this.code = r.data.data.captcha
            })
            .catch(r => {
              console.log(r)
            })
          this.timer = setInterval(() => {
            if (this.second > 0 && this.second <= 60) {
              this.second--
            } else {
              this.show = true
              clearInterval(this.timer)
              this.timer = null
            }
          }, 1000)
        }
      }
    },
    check () {
      let regP = /^[1]([3-9])[0-9]{9}$/
      console.log(this.phonenum)
      if (this.phonenum !== '') {
        if (!regP.test(this.phonenum)) {
          this.$message('手机号格式不正确')
        } else {
          this.flag = true
        }
      }
    },
    async submit () {
      console.log(this.phonenum, this.prefix_num, this.code, this.password)
      if (this.flag !== true && this.code.length !== 4) {
        this.$message('信息必须完整且合法！')
        return false
      }
      if (this.password && this.password.length >= 6) {
        await this.$ajax.post('https://easy-mock.com/mock/5b2385e3debe3c5977248a16/wscn/submit',
          {'phone': this.phonenum, 'captcha': this.code}
        ).then(r => {
          console.log(r)
          let status = r.data.data.success
          if (status === true) {
            this.$message(r.data.data.messgae)
          }
        }).catch(r => {
          console.log(r)
        })
      } else {
        this.$message('密码不符合要求')
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.login {
  margin: 0 auto;
}
.item {
    display: flex;
    margin: 10px 0;
}
.item span {
  margin-left:5px;
}
.item .el-select {
  margin-right:5px;
}
.el-col {
  transform: translateX(-50%);
  margin: 0 50%;
}
</style>
