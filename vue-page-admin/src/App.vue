<template>
  <div id="main">
    <h2>{{ place }}</h2>
     <p>{{day}}</p>
    
    <div style="border: 1px solid #ccc; padding: 10px; border-radius: 10px;">
        <p>选择队伍：</p>
        <el-radio-group v-model="type" @change="getReadyList">
          <el-radio :label="type1">{{type1}} </el-radio>
          <el-radio :label="type2">{{type2}} </el-radio>
        </el-radio-group>

       <el-divider></el-divider>

        <p >
          已完成 {{done}} 人 / 总预约 {{total}} 人
          <el-progress 
            style="width: 80%; text-align: center; margin: 0 auto;" 
            :text-inside="true" :stroke-width="24" :percentage="doneP"></el-progress>
        </p>
    </div>

  

     <el-divider></el-divider>
    <div>
       <!-- <el-divider content-position="right">{{readySize}}人准备检查，开始检录</el-divider> -->
      <p>{{readySize}}人准备检查，开始检录</p>
      <!-- <el-button @click="getReadyList">{{readySize}}人准备检查，开始检录</el-button> -->
      <div class="wait-box" v-for="(item, index) in readyList" :key="item.id">
        <p >
          <span style="color: red; font-weight: bold; fron-size:16pt;">{{ item.id}} </span>
          <span style="line-height: 1em; margin-right: 10px;"> {{ item.name}} {{amount}}个人  </span>
        </p>
         
        <el-row>
          <el-col :span="18">
            <el-input type="text" minlength="4" maxlength="4" v-model="checkList['code' + index]">
              <el-button slot="append" style="color: #409EFF;font-size: 14pt;" 
                @click="setCheck(item.id, item.code4, checkList['code' + index], index)">检录</el-button>
            </el-input>
          </el-col>
          <el-col :span="6">
            <el-button type="danger" icon="el-icon-close" circle @click="expireOrder(item.id)"></el-button>
          </el-col>
        </el-row>
           
      </div>
    </div>

   

    <footer style="margin-top:200px">
      技术支持： 微信 hygkui  手机 15009281751
    </footer>
  </div>
</template>

<script>
const axios = require("axios");
const baseUrl = 'api'
export default {
  data() {
    return {
      readySize: 3,
      code4: '',
      checkList: {
        code0: '',
        code1: '',
        code2: '',
        code3: '',
        code4: '',
      },
      done: 0,
      total: 1,
      timer: null,
      day: '',
      place: '富力城南区检测点',
      type1: '带老人小孩',
      type2: '个人一码通',
      type: '带老人小孩',
      readyList: null,
    }
  },
  computed: {
    doneP() {
      return Math.floor(this.done * 100 / this.total)
    }
  },
  methods: {
    async getReadyList() {
      this.readyList = null;
      const result = await axios.request({
        url: `${baseUrl}/orders/getReadyList`,
        method: 'GET',
        params: {
          type: this.type,
          day: this.day,
          size: this.readySize,
        }
      })
      const { readyList, done, total } = result.data; 
      this.readyList = readyList;
      this.done = done;
      this.total = total;
    },
    async setCheck(id, code4, _code4, index) {
      if (code4 === _code4) {
        await axios.request({
          url: `${baseUrl}/orders/${id}`,
          method: 'PUT',
          data: {
            status: 1
          }
       })
        this.$message({
          message: '检录成功',
          type: 'success'
        })
        this.checkList['code'+index] = '';
       await this.getReadyList();
      } else {
        this.$message.error('请检查预约信息手机号后四位');
      }
    },
    async expireOrder(id) {
       this.$confirm('废弃预约号码, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true,
        }).then(async () => {
          await axios.request({
            url: `${baseUrl}/orders/${id}`,
            method: 'PUT',
            data: {
              status: 3
            }
          })
          this.$message({
            type: 'success',
            message: '已废弃!'
          });
          await this.getReadyList();
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });          
        });
    }
  },
  async mounted() {
    this.day = new Date().toLocaleDateString();
    this.getReadyList();
  }
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
  /* border: 1px solid red; */
  padding: 5px;
  border-radius: 5px;
  font-size: 16pt;
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

.wait-box {
  clear: both;
  width: 100%;
   box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04);
   margin: 20px 0;
   padding: 10px;
}
  
</style>
