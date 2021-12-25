<template>
  <div id="main">
    <h2>{{ place }}</h2>
    <p v-if="userData">{{ userData.day }}</p>
    <div class="notice">
      <p>
        排队保持一米安全距离，全程佩戴好口罩，咳嗽、打喷嚏时用纸巾或手肘遮挡，避免飞沫传播风险。
        老人或小孩没有健康码的，请携带身份证或户口本现场进行登记。
      </p>
    </div>

    <div class="status" v-if="isTodayReady">
      <p style="font-size: 18pt; font-weight: bold">
        前面还有
        <span style="color: red">{{ waitAmount }}</span>
        人排队
      </p>
      <el-card shadow="always" class="box-card">
        <p>
          号码
          <span style="font-size: 18pt; color: red; font-weight: bold">{{
            userData.id
          }}</span>
        </p>
        <p style="font-size: 14pt">{{ userData.name }}</p>
        <p>手机后四位： {{ userData.code4 }}</p>
      </el-card>
      <br />
      <el-button type="danger" @click="clearOrder">重新预约</el-button>
      <br />
      <br />
      <hr />
      <br />
      <p>最前30名</p>
      <el-table :data="waitList" stripe style="width: 100%">
        <el-table-column prop="id" label="号码" width="180"> </el-table-column>
        <el-table-column prop="name" label="姓名" width="180">
        </el-table-column>
        <el-table-column prop="type" label="队伍" width="180">
        </el-table-column>
        <el-table-column label="状态" width="180">
          <template slot-scope="scope">
            <span style="margin-left: 10px">{{
              ['已预约', '现场检录', '已完成', '已作废'][scope.row.status]
            }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="amount" label="人数"> </el-table-column>
      </el-table>
    </div>

    <div class="apply" v-else>
      <p>选择队伍：</p>
      <el-radio-group v-model="type">
        <el-radio :label="type1">{{ type1 }} </el-radio>
        <el-radio :label="type2">{{ type2 }} </el-radio>
      </el-radio-group>

      <p v-if="type == ''">
        拥有一码通/健康码，自己来做核酸检测，请选择“个人一码通”队伍预约排队。
      </p>

      <div v-if="type == type1" class="apply-from">
        <div class="grid-content bg-purple">
          <el-form :model="userData" label-width="100px">
            <el-form-item label="名字">
              <el-input v-model="userData.name"></el-input>
            </el-form-item>
            <el-form-item label="手机号后四位">
              <el-input
                v-model="userData.code4"
                maxlength="4"
                minlength="4"
              ></el-input>
            </el-form-item>
            <el-form-item label="人数">
              <el-radio-group v-model="userData.amount">
                <el-radio :label="2">2人</el-radio>
                <el-radio :label="3">3人</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-form>
          <el-button type="primary" round @click="order1">取 号</el-button>
        </div>
      </div>
      <div v-if="type == type2" class="apply-from">
        <div class="grid-content bg-purple-light">
          <el-form :model="userData" label-width="100px">
            <el-form-item label="名字">
              <el-input v-model="userData.name"></el-input>
            </el-form-item>
            <el-form-item label="手机号后四位">
              <el-input
                v-model="userData.code4"
                maxlength="4"
                minlength="4"
              ></el-input>
            </el-form-item>
          </el-form>
          <el-button type="primary" round @click="order2">取 号</el-button>
        </div>
      </div>
    </div>

    <footer style="margin-top: 200px">
      技术支持:微信 hygkui 15009281751
      <p>&copy;2021 航天基地富力城小区 居民</p>
    </footer>
  </div>
</template>

<script>
const axios = require('axios')
const baseUrl = 'api'

const defaultUserData = {
  id: 0,
  name: '',
  code4: '',
  amount: 1,
  type: 0,
  status: 0,
  day: '',
}
export default {
  data() {
    return {
      timer: null,
      place: '富力城南区检测点',
      notice: ``,
      type: '',
      type1: '带老人小孩',
      type2: '个人一码通',
      isTodayReady: false,
      waitAmount: 233,
      userData: defaultUserData,
      waitList: null,
      readyList: null,
    }
  },
  methods: {
    startHacking() {
      this.$notify({
        title: 'It works!',
        type: 'success',
        message:
          "We've laid the ground work for you. It's time for you to build something epic!",
        duration: 5000,
      })
    },
    async order1() {
      const { name, amount, code4, status } = this.userData
      const type = this.type1
      const day = new Date().toLocaleDateString()
      await this.createOrder({
        name,
        amount,
        code4,
        type,
        day,
        status,
      })
    },
    async order2() {
      const { name, code4, status } = this.userData
      const amount = 1
      const type = this.type2
      const day = new Date().toLocaleDateString()
      await this.createOrder({
        name,
        amount,
        code4,
        type,
        day,
        status,
      })
    },
    async createOrder(data) {
      const result = await axios.request({
        url: `${baseUrl}/orders`,
        method: 'POST',
        data,
      })
      await localStorage.setItem('userData', JSON.stringify(result.data))
      this.userData = result.data
      this.isTodayReady = true
      await this.fetchList()
    },
    async clearOrder() {
      // clear localstorage
      // clear data
      localStorage.removeItem('userData')
      const result = await axios.put(`${baseUrl}/orders/${this.userData.id}`, {
        status: 3,
      })
      console.log('clearOrder result :>> ', result)
      this.userData.name = ''
      this.userData.amount = 1
      this.userData.code4 = ''
      this.isTodayReady = false
      this.waitList = null
      this.timer && clearInterval(this.timer)
    },
    async fetchList() {
      const { id, type, day } = this.userData
      const result = await axios.request({
        url: `${baseUrl}/orders/getLatest`,
        method: 'GET',
        params: { id, type, day },
      })
      console.log('fetch list result :>> ', result)
      const { waitAmount, waitList } = result.data
      this.waitList = waitList
      this.waitAmount = waitAmount
    },
    async get3Ready() {
      const result = await axios.request({
        url: `${baseUrl}/orders/get3Ready`,
        method: 'GET',
        params: {
          type: this.userData.type,
          day: this.userData.day,
        },
      })
      console.log('get3Ready list result :>> ', result)
      this.readyList = result.data
    },
  },
  async mounted() {
    let data = await localStorage.getItem('userData')
    let _userData = JSON.parse(data)
    if (_userData.day === new Date().toLocaleDateString()) {
      console.log('match day :>> ')
      this.isTodayReady = true
      this.userData = _userData
      await this.fetchList()
    } else {
      this.isTodayReady = false
      this.userData = defaultUserData
    }

    if (this.timer) {
      clearInterval(this.timer)
    }
    this.timer = setInterval(this.fetchList, 30 * 1000)
  },
}
</script>

<style>
#main {
  font-family: Helvetica, sans-serif;
  text-align: center;
  /* background-color: #000;
  color: #fff; */
  padding: 20px;
  margin: 10px auto;
}

.notice {
  color: red;
  font-weight: bold;
  border: 1px solid gray;
  padding: 5px;
  border-radius: 5px;
  font-size: 12pt;
}

.userdata-form {
  width: 300px;
  margin: 10px auto;
  padding: 5px;
  font-size: 14pt;
  clear: both;
}

.box-card {
  margin: 0 auto;
  text-align: center;
}

form {
  margin: 20px;
}

.el-table th {
  text-align: center;
  font-weight: bold;
}

.apply-from {
  background-color: #eee;
  margin: 10px auto;
  padding: 10px 10px 30px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
}

.el-radio__label {
  font-size: 16pt;
}
</style>
