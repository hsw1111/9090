<template>
	<div style="margin-right:20px;">
		<div id="earIncom_header">
      <div class="earD_con">
        <div class="timeSelectBtn">
          <el-button @click='getAllDate' class="active">所有明细</el-button>
          <el-button @click='getMonthDate'>本月明细</el-button>
          <el-button @click='getDailyDate'>今日明细</el-button>
          <el-button @click='handleChangeType'>指定时间段</el-button>
        </div>
        <el-date-picker style="vertical-align: middle;" v-show="show" type="datetimerange" :picker-options="pickerOptions2" placeholder="选择时间范围" align="right">
        </el-date-picker>
        <el-button style="vertical-align: middle;" icon="search" class="timeSelect_button">查询</el-button>        
      </div>
		</div>

		<div id="earD_excel">
			<div id="earD_all">
				<h1>
          <el-tooltip class="item" effect="dark" content="所有车辆骑行收益" placement="bottom-end"> 
					  <p>合计：<span>30000元</span></p>
          </el-tooltip>
					<p @click='export_excel'>导出明细到Excel</p>
				</h1>
			</div>
		</div>

		<div id="earD_body">
			<el-table
      :data="tableData"
      style="width: 100%">
      <el-table-column
        prop="money"
        label="金额"
        min-width="60">
      </el-table-column>
      <el-table-column
        prop="bikeCode"
        label="车牌号"
        min-width="70">
      </el-table-column>
      <el-table-column
        prop="placeOrderTime"
        label="消费时间"
        min-width="70">
      </el-table-column>
      <el-table-column
        prop="journey"
        label="里程"
        min-width="70">
      </el-table-column>
      </el-table-column>
      <el-table-column
        prop="orderDate"
        label="订单日期">
      </el-table-column>
    </el-table>
		</div>

		<div id="earD_page">
			<div class="M-box">
			</div>
		</div>
	</div>
</template>

<style scoped>
  html,body,h1,h2,h3,h4,h5,h6,td,tr,th,table,thead,tbody,dl,dt,dd,p,span,ul,li,ol {
  	margin: 0;
  	padding: 0;
  }

  body {
  	font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
  }

  .loading_class {
    background: rgba(68,68,68,0.4);
  }

	#earIncom_header {
    /*width: 100%;*/
    height: 72px;
    line-height: 52px;
    background: #fff;
    border: 1px solid #e7ecf1;
    padding: 20px 30px 20px 30px;
	}

  #earIncom_header .earD_con {
    background: #f3f0f0;
    padding: 10px;
  }

	/*#earIncom_header button {
		width: 100px;
    height: 30px;
		font-size: 12px;
    margin-right: 10px;
    cursor: pointer;
    background-color: #fff;
    border-radius: 4px;
		outline: none;
    color: #878787;
    border: 1px solid #cecece;
    transition: all .2s linear 0s;
	}

	#earIncom_header button:nth-of-type(1) {
		margin-left: 30px;
	}*/

	/*#earIncom_header button:hover {
    background: rgb(66,66,66);
    border: 1px solid rgb(66,66,66);
		color: #fff;
	}*/

	/*#earIncom_header button.active {
    font-size: 12px;
    color: #fff;
    background: rgba(66,66,66, 0.8);
    border: 1px solid rgb(66,66,66);
	}*/

  .el-table td:nth-of-type(1) .cell {
    color: #f60;
  }

  div.timeSelectBtn {
    display:block;
    float:left;
    margin-right: 8px;
    padding-left: 20px;
  }

  div.timeSelectBtn button {
    margin-left: 8px;
    display: inline-block;
    border: none;
    outline: none;
    font-size: 12px;
    color: #fff;
    background: rgba(66,66,66, 0.8);
    transition: all .2s linear 0s;
    box-sizing: border-box;
    height:35px;
  }

  div.timeSelectBtn button.active {
      background: rgb(66,66,66);
      color:#fff
  }
  div.timeSelectBtn button:hover{color:#fff}

	#earD_excel {
    padding: 10px 30px 0px 30px;
		background:#fff;
    border: 1px solid #e7ecf1;
    border-bottom: none;
    margin-bottom: -10px;
    margin-top: 20px;
	}

	#earD_all {
		width: 100%;
		height: 40px;
		background: #fcfcd3;
		margin-top: 20px;
		margin-bottom: 10px;
	}

	#earD_all h1 {
		width: 100%;
		line-height: 40px;
		height: 40px;
	}

	#earD_all h1 p:nth-of-type(1) {
		float: left;
		margin-left: 10px;
		font-size: 12px;
		color: #868671;
    cursor: pointer;
	}

	#earD_all h1 p:nth-of-type(2) {
		float: right;
		font-size: 12px;
		margin-right: 10px;
		color: #ff9900;
	}

	#earD_all h1 p:nth-of-type(2):hover {
		text-decoration: underline;
    cursor: pointer;
	}

	#earD_body {
    padding: 20px 20px 10px 20px;
    background: #fff;
    /*margin-top: 20px;*/
    padding-left: 28px;
    padding-right: 28px;
    border: 1px solid #e7ecf1;
    border-bottom: none;
    border-top: none;
	}

  #earD_body tr.el-table__row div.cell {
    color: #f60;
  }

	#earD_page {
    padding: 4px 10px 0 18px;
		padding-bottom: 100px;
		background: #fff;
		border: 1px solid #e7ecf1;
		border-top: none;
		min-height: 233px;
	}

</style>

<script>
// import $ from 'jquery'
import request from 'superagent'
import moment from 'moment'
// require('../../../assets/lib/js/exportExcel.js')
//require('../../../assets/lib/js/Blob.js')
import { host } from '../../../config/index.js'
export default {
  data () {
    return {
      tableData: [],
      timer: null,
      totalPage: '',
      pickerOptions2: {
        shortcuts: [
          {
            text: '最近一周',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
              picker.$emit('pick', [start, end])
            }
          },
          {
            text: '最近一个月',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
              picker.$emit('pick', [start, end])
            }
          },
          {
            text: '最近三个月',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
              picker.$emit('pick', [start, end])
            }
          }
        ]
      },
      time_line: '',
      show: false
    }
  },
  mounted () {
   
    this.$router.push('/index/earningsDetail?type=getAllRevenue')
    request
      .post(host + 'franchisee/revenue/getAllRevenue')
      .withCredentials()
      .set({
        'content-type': 'application/x-www-form-urlencoded'
      })
      .send({
        'franchiseeId': '123456',
        'userId': 'admin'
      })
      .end((err, res) => {
        if (err) {
          console.log('err:' + err)
        } else {
          // console.log(res)
          // console.log(JSON.parse(res.text).list)
          var newArr = JSON.parse(res.text).list
          var pageNumber = JSON.parse(res.text).totalPage
          this.totalPage = pageNumber
          var arr2 = this.tableDataDel(newArr)
          this.$store.dispatch('earningsDate_action', { arr2 })
          this.tableData = this.$store.state.earningsDate.arr2
          $('.M-box').pagination({
            pageCount: pageNumber,
            jump: true,
            coping: true,
            homePage: '首页',
            endPage: '尾页',
            prevContent: '«',
            nextContent: '»'
          })
        }
      })

    // 点击切换查看类型
    $('#earIncom_header button').click('button', function () {
      $('button.active').removeClass('active')
      $(this).addClass('active')
    })
    var that = this
    $('.timeSelectBtn button').on('click', function (e) {
      that.handleChangeType(e)
    })
  },
  beforeUpdate () {
    var that = this
    $('.M-box').click('a', function (e) {
      // console.log(e)
      that.pageUpdate(e)
    })
  },
  methods: {
    handleChangeType (e) {
      // console.log(e.currentTarget.innerText)
      if (e.currentTarget.innerText === '指定时间段') {
        this.show = true
      } else {
        this.show = false
      }
    },
    export_excel () {
      this.$confirm('确认导出吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
      .then(() => {
        require.ensure([], () => {
          this.$loading({customClass: 'loading_class'})
          var that = this
          setTimeout(() => {
            that.$loading({customClass: 'loading_class'}).close()
            const { export_json_to_excel } = require('../../../assets/lib/js/Export2Excel.js')
            const tHeader = ['金额', '车牌号', '消费时间', '里程', '订单日期']
            const filterVal = ['money', 'bikeCode', 'placeOrderTime', 'journey', 'orderDate']
            const list = this.tableData
            var type = that.$route.query.type
            request
              .post('url?type=' + type)
              .send({
                'franchiseeId': '123456',
                'userId': 'admin'
              })
              .end((err, res) => {
                if (err) {
                  console.log('err:' + err)
                } else {
                  // 数据处理
                  const data = this.formatJson(filterVal, list)
                  export_json_to_excel(tHeader, data, '列表excel')
                  that.$message({
                    type: 'success',
                    message: '导出成功'
                  })
                }
              })
          }, 1000)
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消'
        })
      })
    },
    formatJson (filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    },
    tableDataDel (arr) {
      var arrDeled = []
      for (var i = 0; i < arr.length; i++) {
        var obj = {}
        obj.money = arr[i].money
        obj.bikeCode = arr[i].bikeCode
        obj.placeOrderTime = Math.floor((arr[i].time) / 60000) + ' 分钟'
        obj.journey = arr[i].mileage
        obj.orderDate = moment(arr[i].chargeTime).format('YYYY-MM-DD')
        arrDeled.push(obj)
      }

      return arrDeled
    },
    getAllDate () {
      this.$router.push('/index/earningsDetail?type=getAllRevenue')
    },
    getMonthDate () {
      this.$router.push('/index/earningsDetail?type=getRevenueCurMonth')
      request
        .post(host + 'franchisee/revenue/getRevenueCurMonth')
        .withCredentials()
        .set({
          'content-type': 'application/x-www-form-urlencoded'
        })
        .send({
          'franchiseeId': '123456',
          'userId': 'admin'
        })
        .end((err, res) => {
          if (err) {
            console.log('err:' + err)
          } else {
            // console.log(res)
            var newArr = JSON.parse(res.text).list
            var pageNumber = JSON.parse(res.text).totalPage
            var arr2 = this.tableDataDel(newArr)
            this.totalPage = pageNumber
            this.$store.dispatch('earningsDate_action', { arr2 })
            this.tableData = this.$store.state.earningsDate.arr2
            $('.M-box').pagination({
              pageCount: pageNumber,
              jump: true,
              coping: true,
              homePage: '首页',
              endPage: '尾页',
              prevContent: '«',
              nextContent: '»'
            })
          }
        })
    },
    getDailyDate () {
      this.$router.push('/index/earningsDetail?type=getRevenueCurDay')
      request
        .post(host + 'franchisee/revenue/getRevenueCurDay')
        .withCredentials()
        .set({
          'content-type': 'application/x-www-form-urlencoded'
        })
        .withCredentials()
        .set({
          'content-type': 'application/x-www-form-urlencoded'
        })
        .send({
          'franchiseeId': '123456',
          'userId': 'admin'
        })
        .end((err, res) => {
          if (err) {
            console.log('err:' + err)
          } else {
            var newArr = JSON.parse(res.text).list
            var pageNumber = JSON.parse(res.text).totalPage
            var arr2 = this.tableDataDel(newArr)
            this.totalPage = pageNumber
            this.$store.dispatch('earningsDate_action', { arr2 })
            this.tableData = this.$store.state.earningsDate.arr2
            $('.M-box').pagination({
              pageCount: pageNumber,
              jump: true,
              coping: true,
              homePage: '首页',
              endPage: '尾页',
              prevContent: '«',
              nextContent: '»'
            })
          }
        })
    },
    pageUpdate (e) {
      var that = this
      clearTimeout(this.timer)
      if (e.target.tagName === 'A' || e.target.tagName === 'SPAN') {
        if (e.target.innerHTML === '首页') {
          e.target.innerHTML = 1
        } else if (e.target.innerHTML === '尾页') {
          e.target.innerHTML = this.totalPage
        } else if (e.target.innerHTML === '«') {
          e.target.innerHTML = Number($('.M-box span.active')[0].innerHTML) - 1
        } else if (e.target.innerHTML === '»') {
          e.target.innerHTML = Number($('.M-box span.active')[0].innerHTML) + 1
        } else if (e.target.innerHTML === '...') {
          return
        }
      } else {
        return
      }
      var type = this.$route.query.type
      this.timer = setTimeout(function () {
        request
          .post(host + 'franchisee/revenue/' + type + '?page=' + e.target.innerHTML)
          .withCredentials()
          .set({
            'content-type': 'application/x-www-form-urlencoded'
          })
          .send({
            'franchiseeId': '123456',
            'userId': 'admin'
          })
          .end((error, res) => {
            if (error) {
              console.log('error:', error)
            } else {
              var pagedata = (JSON.parse(res.text)).list
              var arr2 = that.tableDataDel(pagedata)
              that.$store.dispatch('earningsDate_action', { arr2 })
              that.tableData = that.$store.state.earningsDate.arr2
            }
          })
      }, 200)
    },
    dataUpdate () {
      var type = this.$route.query.type
      request
        .post(host + 'franchisee/revenue/' + type)
        .withCredentials()
        .set({
          'content-type': 'application/x-www-form-urlencoded'
        })
        .send({
          'franchiseeId': '123456',
          'userId': 'admin'
        })
        .end((error, res) => {
          if (error) {
            console.log('error:', error)
          } else {
            // console.log(JSON.parse(res.text))
            var pagedata = (JSON.parse(res.text)).list
            var pageNumber = JSON.parse(res.text).totalPage
            var arr2 = this.tableDataDel(pagedata)
            this.totalPage = pageNumber
            this.$store.dispatch('earningsDate_action', { arr2 })
            this.tableData = this.$store.state.earningsDate.arr2
          }
        })
    }
  },
  created () {
    this.dataUpdate()
  },
  watch: {
    '$route': 'dataUpdate'
  }
}
</script>
