<template>
  <div style="margin-right:20px;">
    <div id="am_search">
      <label>
        <el-form :inline="true" class="demo-form-inline">
          <el-form-item label="角色名称">
            <!-- <el-input v-model="roleName" v-on:blur="initRole" placeholder="请输入角色名称"></el-input> -->
            <el-input v-model="roleName"  placeholder="请输入角色名称"></el-input>
          </el-form-item>
        </el-form>
      </label>
      <el-button @click="queryRole" id="roleSearchBtn">查询</el-button>
    </div>
  
    <!-- account -->
    <div class="account">
      <h1>
        <button type="button" @click="openAddRole">添加角色</button>
        <el-dialog class="addRole" title="添加角色" :visible.sync="dialogFormVisible" :modal-append-to-body="false" :modal="true">
          <el-form :model="form" :rules="rules" ref="ruleForm">
            <el-form-item label="角色名称" prop="roleName" class="rolename" :label-width="formLabelWidth">
              <el-input v-model="form.roleName" placeholder="请输入角色名称"></el-input>
            </el-form-item>
            <el-form-item label="备注" prop="description" class="rolename" :label-width="formLabelWidth">
              <el-input type="textarea" v-model="form.description"></el-input>
            </el-form-item>
            <el-form-item label="权限列表" class="rolename" prop='rolePowerList' :label-width="formLabelWidth">
              <el-tree :data="form.rolePowerList" show-checkbox ref="tree" node-key="id" :props="defaultProps">
              </el-tree>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer addfooter">
            <el-button class="addRoleBtn" type="primary" @click="handleAddRole">立即创建</el-button>
            <el-button class="addRoleBtn" @click="closeAddRole">取消</el-button>
          </div>
        </el-dialog>
      </h1>
  
      <!-- 表单 -->
      <el-table :data="tableData" style="width: 100%; font-size:13px;" element-loading-text="拼命加载中" v-loading="loading">
        <el-table-column prop="roleName" label="角色名称" min-width="160"></el-table-column>
        <el-table-column prop="description" label="备注" min-width="160"></el-table-column>
        <el-table-column label="包含用户" min-width="170">
          <template slot-scope="scope">
            <ul class="roleList">
              <el-tag v-for="name in scope.row.adminUserList" :key="name" type="gray">
                {{name}}
              </el-tag>
              <!-- <li :key="name" v-for="name of scope.row.adminUserList">{{name}}</li> -->
            </ul>
          </template>
        </el-table-column>
        <el-table-column prop="del" label="操作">
          <template slot-scope="scope">
            <i class="el-icon-edit" style="margin-right:10px;" @click="openEditRole(scope)" title="修改"></i>
            <i class="el-icon-close" title="删除" @click="handleDeleteRole(scope)"></i>
          </template>
        </el-table-column>
      </el-table>

      <el-dialog id="err_form" class="edit_role" v-loading="loading2" title="修改角色" :visible.sync="dialogEditVisible" :modal-append-to-body="false" :modal="true" :show-message="true">
        <el-form :model="editForm" :rules="rules1" ref="editForm">
          <el-form-item label="角色名称" class="rolename" prop="roleName" :label-width="formLabelWidth" >
            <el-input v-model="editForm.roleName" placeholder="请输入角色名称"></el-input>
          </el-form-item>
          <el-form-item label="备注" prop="description" :label-width="formLabelWidth">
            <el-input type="textarea" v-model="editForm.description"></el-input>
          </el-form-item>
          <el-form-item label="权限列表"  prop='rolePowerList' :label-width="formLabelWidth">
            <el-tree ref="tree"  :data="editForm.rolePowerList" show-checkbox node-key="id" :props="defaultProps2" :default-checked-keys="fathCode">
            </el-tree>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer editfooter">
  
          <el-button class="eidtRoleBtn" type="primary" @click="handleEditRole">确定</el-button>
          <el-button class="eidtRoleBtn" @click="closeEditRole">取消</el-button>
        </div>
      </el-dialog>
      <el-pagination style="margin-left: 0px;" @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="currentPage3" :page-size="10" layout="prev, pager, next, jumper" v-show="pageShow" :total="totalItems">
      </el-pagination>
    </div>
  
    <!--<div v-show='router_show' >-->
    <router-view id="account_router"></router-view>
    <!--</div>-->
  </div>
</template>

<script>
// import $ from 'jquery'
import request from 'superagent'
import { host } from '../../../config/index'
export default {
  data() {
    var validateRoleName = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入角色名'));
      } else {
        callback();
      }
    }
     var validateRoleName1 = (rule, value, callback) => {
      if (this.editForm.roleName === '') {
        callback(new Error('请输入角色名'));
      } else {
        callback();
      }
    }
    return {
      rowMenuList: [],
      isQuery: false,
      currentPage3: 1,
      totalItems: 1,
      pageShow: false,
      roleName: '',
      loading: false,
      loading2: false,
      input: '',
      dialogFormVisible: false,
      dialogEditVisible: false,
      currentPage: 1,
      totalPage: 1,
      tableData: [],
      initData: [],
      fathCode: [],
      childrenCode: [],
      router_show: false,
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      defaultProps2: {
        children: 'children',
        label: 'label'
      },
      form: {
        roleName: '',
        description: '',
        rolePowerList: [
          {
            id: 10,
            label: '首页'
          },
          {
            id: 20,
            label: '加盟商管理'
          },
          {
            id: 30,
            label: '车辆管理'
          },
          {
            id: 40,
            label: '订单管理'
          },
          {
            id: 50,
            label: '报表管理',
            children: [
              {
                id: 51,
                label: '收益排行'
              }
              // {
              //   id: 52,
              //   label: '24小时数据走势'
              // },
              // {
              //   id: 53,
              //   label: '热力图'
              // },
              // {
              //   id: 54,
              //   label: '异常数据'
              // }
            ]
          },
          {
            id: 60,
            label: '结算管理'
          },
          {
            id: 70,
            label: '个人中心'
          },
          {
            id: 80,
            label: '账号管理'
          },
          {
            id: 90,
            label: '角色管理'
          },
          // {
          //   id: 1800,
          //   label: '信息中心'
          // },
          // {
          //   id: 1900,
          //   label: '日志管理',
          //   children: [
          //     {
          //       id: 1901,
          //       label: '登录日志'
          //     },
          //     {
          //       id: 1902,
          //       label: '操作日志'
          //     },
          //   ]
          // }
        ],
      },
      formLabelWidth: '120px',
      editForm: {
        roleName: '',
        description: '',
        id: '',
        index: '',
        roleType: '',
        rolePowerList: [
          {
            id: 10,
            label: '首页'
          },
          {
            id: 20,
            label: '加盟商管理'
          },
          {
            id: 30,
            label: '车辆管理'
          },
          {
            id: 40,
            label: '订单管理'
          },
          {
            id: 50,
            label: '报表管理',
            children: [
              {
                id: 51,
                label: '收益排行'
              }
              // {
              //   id: 1302,
              //   label: '24小时数据走势'
              // },
              // {
              //   id: 1303,
              //   label: '热力图'
              // },
              // {
              //   id: 1304,
              //   label: '异常数据'
              // }
            ]
          },
          {
            id: 60,
            label: '结算管理'
          },
          {
            id: 70,
            label: '个人中心'
          },
          {
            id: 80,
            label: '账号管理'
          },
          {
            id: 90,
            label: '角色管理'
          },
          // {
          //   id: 1800,
          //   label: '信息中心'
          // },
          // {
          //   id: 1900,
          //   label: '日志管理',
          //   children: [
          //     {
          //       id: 1901,
          //       label: '登录日志'
          //     },
          //     {
          //       id: 1902,
          //       label: '操作日志'
          //     },
          //   ]
          // }
        ]
      },
      rules: {
        roleName: [
          { validator: validateRoleName, trigger: 'blur', required: true }
        ]
      },
       rules1: {
        roleName: [
          { validator: validateRoleName1, trigger: 'blur', required: true }
        ]
      },
      flag: false,
      isSearch: false
    }
  },
  mounted() {
    $(".sign").removeClass('is-active')
    $('.sign[name="90"]').addClass('is-active')
    document.title = '角色管理'
    this.loadRole()
    var roleList = window.localStorage.getItem("admin_authList")
  },
  methods: {
    setCheckedKeys(){
      this.$refs.tree.setCheckedKeys(this.rowMenuList)
    },
    handleSizeChange(val) {
      // console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      // 初始化查询
      this.loading = true
      request
      .post(host + 'beepartner/admin/Role/findAdminRole')
      .withCredentials()
      .set({
        'content-type': 'application/x-www-form-urlencoded'
      })
      .send({
        'currentPage': val,
        'roleName': this.isSearch===false?'':this.roleName
      })
      .end((err, res) => {
        if (err) {
          console.log(err)
          this.loading = false
        } else {
          this.checkLogin(res)
          this.loading = false
          var result = JSON.parse(res.text).data
          var totalPage = Number(JSON.parse(res.text).totalPage)
          var newArr = result.map((item)=>{
            var arr = item.adminUserList.map((item)=>{
              return item.userName
            })
           return Object.assign({},item,{adminUserList: arr})
         })
          if (totalPage > 1) {
            this.pageShow = true
          } else {
            this.pageShow = false
          }
          this.totalItems = Number(JSON.parse(res.text).totalItems)
          this.tableData = newArr
          this.initData = this.tableData
        }
      })
    },
    // 无需失去焦点自动查询，以点击查询为主
    // initRole() {

    //   this.isSearch = false

    //   this.isQuery = false
    //   this.currentPage3 = 1
    //   if (this.roleName.trim().length === 0 && this.isQuery === false) {
    //     var that = this
    //     request
    //       .post(host + 'beepartner/admin/Role/findAdminRole')
    //       .withCredentials()
    //       .set({
    //         'content-type': 'application/x-www-form-urlencoded'
    //       })
    //       .end((err, res) => {
    //         if (err) {
    //           console.log(err)
    //         } else {
    //           this.checkLogin(res)
    //           this.loading = false
    //           var result = JSON.parse(res.text).data
    //           var totalPage = Number(JSON.parse(res.text).totalPage)
    //           var newArr = result.map((item)=>{
    //             var arr = item.adminUserList.map((item)=>{
    //               return item.userName
    //             })
    //           return Object.assign({},item,{adminUserList: arr})
    //           })
    //           if (totalPage > 1) {
    //             this.pageShow = true
    //           } else {
    //             this.pageShow = false
    //           }
    //           this.totalItems = Number(JSON.parse(res.text).totalItems)
    //           this.tableData = newArr
    //           this.initData = this.tableData
    //         }
    //       })
    //   }
    // },
    closeAddRole () {
      this.$refs.ruleForm.resetFields()
      this.dialogFormVisible = false
    },
    closeEditRole(){
      // this.$refs.editForm.resetFields()
      this.dialogEditVisible = false
    },
    queryRole() {

      this.isSearch = true

      var that = this
      // if (this.roleName.trim().length !== 0) {
        this.isQuery = true
        that.loading = true
        request
          .post(host + 'beepartner/admin/Role/findAdminRole')
          .withCredentials()
          .set({
            'content-type': 'application/x-www-form-urlencoded'
          })
          .send({
            roleName: this.roleName.trim()
          })
          .end(function (error, res) {
            if (error) {
              console.log(error)
              that.loading = false
            } else {
              that.checkLogin(res)
              that.loading = false
              var result = JSON.parse(res.text).data
              var totalPage = Number(JSON.parse(res.text).totalPage)
              var newArr = result.map((item) => {
                var arr = item.adminUserList.map((item)=>{
                  return item.userName
                })
              return Object.assign({},item,{adminUserList: arr})
              })
              if (totalPage > 1) {
                that.pageShow = true
              } else {
                that.pageShow = false
              }
              that.totalItems = Number(JSON.parse(res.text).totalItems)
              that.tableData = newArr
            }
          })
      // } else {
      //   this.$message({
      //     type: 'warning',
      //     message: '查询条件不能为空'
      //   })
      //   return false
      // }
    },
    openAddRole() {
      this.dialogFormVisible = true
      this.flag = false
    },
    openEditRole(scope) {
      this.rowMenuList = scope.row.menuList.map((item)=>{return item*1})
      var that = this;
       setTimeout(function(){
             that.setCheckedKeys()
          },200)
      this.dialogEditVisible = true
      this.editForm.roleName = scope.row.roleName
      this.editForm.description = scope.row.description
      this.editForm.id = scope.row.id
      this.editForm.index = scope.$index
      this.editForm.roleType = scope.row.roleType
      this.fathCode = []
      this.childrenCode = []
    },
    handleEditRole() {
       var that = this
      var authList = this.getCheckedKeys().join('-')
       this.$refs.editForm.validate((valid) => {
        if (valid) {    
      request
        .post(host + 'beepartner/admin/Role/updateRole')
        .withCredentials()
        .set({
          'content-type': 'application/x-www-form-urlencoded'
        })
        .send({
          'id': that.editForm.id,
          'roleName': that.editForm.roleName,
          'description': that.editForm.description,
          'menuStr': authList
        })
        .end(function (err, res) {
          if (err) {
            console.log(err)
            that.$message({
              type: 'error',
              message: '修改失败!'
            })
          } else {
            that.checkLogin(res)
            that.loading2 = false
            var code = JSON.parse(res.text).resultCode
            if (code === 1) {
              that.$message({
                type: 'success',
                message: '修改成功!'
              })
              // that.tableData.splice(that.editForm.index, 1, JSON.parse(JSON.parse(res.text).data))
              // //that.tableData.splice(that.editForm.index,1,{roleName: that.editForm.roleName,description: that.editForm.description, id: that.editForm.id})
              that.flag = true
              that.dialogEditVisible = false
              that.loadRole()
            } else {
              that.$message({
                type: 'error',
                message: '修改失败!'
              })
            }
          }
        })
        }else{
          return false
        }
       })
    },
    handleDeleteRole(scope) {
      var that = this
      if(scope.row.adminUserList.length>0){
        this.$message({
          type:'error',
          message:'该角色下有用户，禁止删除！'
        })
        return;
      }
      
      this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.loading = true
        request
          .post(host + 'beepartner/admin/Role/deleteAdminRole')
          .withCredentials()
          .set({
            'content-type': 'application/x-www-form-urlencoded'
          })
          .send({
            id: scope.row.id
          })
          .end((err, res) => {
            if (err) {
              console.log(err)
              this.loading = flase
              this.$message({
                type: 'error',
                message: '删除失败!'
              })
            } else {
              that.checkLogin(res)
              that.loading = false
              var code = JSON.parse(res.text).resultCode
              if (code === 1) {
                this.$message({
                  type: 'success',
                  message: '删除成功!'
                })
                that.tableData.splice(scope.$index, 1)
              } else {
                this.$message({
                  type: 'error',
                  message: JSON.parse(res.text).message
                })
              }
            }
          })
      }).catch(() => {
        // this.$message({
        //   type: 'info',
        //   message: '已取消删除'
        // })
      })
    },
    handleAddRole() {
      var authList = this.getCheckedKeys().join('-')
      var that = this

      this.$refs.ruleForm.validate((valid) => {
        if (valid) {
          this.dialogFormVisible = false
          request
            .post(host + 'beepartner/admin/Role/addAdminRole')
            .withCredentials()
            .set({
              'content-type': 'application/x-www-form-urlencoded'
            })
            .send({
              description: that.form.description,
              roleName: that.form.roleName,
              menuStr: authList
            })
            .end((err, res) => {
              if (err) {
                console.log(err)
              } else {
                that.checkLogin(res)
                var code = JSON.parse(res.text).resultCode
                if (code === 1) {
                  that.$refs.ruleForm.resetFields()
                  that.$message({
                    type: 'success',
                    message: '恭喜您！添加角色成功'
                  })
                  // var newRole = JSON.parse(JSON.parse(res.text).data)
                  // console.log(newRole)
                  // that.tableData.unshift(newRole)
                  that.flag = true
                } else {
                  that.$message({
                    type: 'error',
                    message: 'sorry！角色已存在，请重新添加'
                  })
                }
              }
            })
        } else {
          // console.log('error submit!!');
          return false;
        }
      })
    },
    getCheckedKeys() {
      return this.$refs.tree.getCheckedKeys()
    },
    loadRole () {
      var that = this
      that.loading = true
      that.pageShow = false
      request
        .post(host + 'beepartner/admin/Role/findAdminRole')
        .withCredentials()
        .set({
          'content-type': 'application/x-www-form-urlencoded'
        })
        .end((err, res) => {
          if (err) {
            console.log(err)
            that.loading = false
          } else {
            that.checkLogin(res)
            that.loading = false
            var result = JSON.parse(res.text).data
            var totalPage = Number(JSON.parse(res.text).totalPage)
            var newArr = result.map((item)=>{
              var arr = item.adminUserList.map((item)=>{
                return item.userName
              })
            return Object.assign({},item,{adminUserList: arr})
          })
            if (totalPage > 1) {
              that.pageShow = true
            } else {
              that.pageShow = false
            }
            that.totalItems = Number(JSON.parse(res.text).totalItems)
            that.tableData = newArr
            that.initData = that.tableData
          }
        })
    },
    checkLogin (res) {
      if (JSON.parse(res.text).message === '用户登录超时') {
          this.$router.push('/login')
      }
    }
  },
  watch: {
    'flag':{
      handler:function(){
        if(this.flag){
          this.loadRole()
        }else{
          this.loading2 = false
          return
        }
      },
      deep:true
    }
  }
}
</script>

<style scoped>
.el-pagination {
  padding-left: 0;
  border-left: 0;
  margin-top: 10px;
  margin-bottom: 0px;
}

html,
body,
h1,
h2,
h3,
h4,
h5,
h6,
td,
tr,
th,
table,
thead,
tbody,
dl,
dt,
dd,
p,
span,
ul,
li,
ol {
  margin: 0;
  padding: 0;
}

body {
  background: #f2f2f2;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
}


/*#account_router_cover {
  width: 100%;
  height: 100%;
  background: rgba(128,128,128,0.6);
  position: fixed;
  left: 0;
  top: 0;
}*/

#account_router {
  width: 100%;
  height: 100%;
  background: rgba(68, 68, 68, 0.6);
  position: fixed;
  z-index: 100;
  left: 0;
  top: 0;
}


/*#account_router {
  position: fixed;
  left: 0;
  top: 0;
  display: block;
  height: 80%;
  margin-left: 10%;
  margin-top: 5%;
  width: 80%;
  z-index: 100;
  background: #f60;
  overflow: hidden;
}*/

div.account {
  /* width: 100%; */
  padding: 0 20px 20px 20px;
  background: #fff;
  margin-top: 20px;
  border: 1px solid #e7ecf1;
  /* border-bottom: none; */
  /* padding-bottom: 100%; */
}

div.account>h1 {
  width: 90%;
  height: 68px;
  line-height: 55px;
}

div.account>h1 button {
  width: 100px;
  height: 30px;
  line-height: 25px;
  display: inline-block;
  border: none;
  outline: none;
  font-size: 14px;
  color: #fff;
  border-radius: 4px;
  background: rgba(66, 66, 66, 0.8);
  transition: all .2s linear 0s;
}

div.account>h1 button:hover {
  background: rgb(66, 66, 66);
  cursor: pointer;
}

#am_search {
  paddint-top:5px;
  width: 100%;
  height: 70px;
  background: #faebd7;
  border: 1px solid #e7ecf1;
}

.account_my_input {
  width: 320px;
  height: 30px;
  outline: none;
  border-radius: 4px;
  margin-top: 4px;
  border: 1px solid #ddd;
  display: inline-block;
}

#am_search label {
    margin-top: 15px;
    margin-left: 21px;
    float: left;
}

#am_search label span {
  line-height: 70px;
  font-weight: 400;
  font-size: 14px;
  display: block;
  margin-right: 10px;
  float: left;
}

#am_search button {
  display: inline-block;
  line-height: 1;
  white-space: nowrap;
  cursor: pointer;
  float: right;
  background: #fff;
  border: 1px solid #c4c4c4;
  color: #1f2d3d;
  margin: 18px 34px;
  padding: 10px 15px;
  border-radius: 4px;
}

#am_search button:hover {
  color: #20a0ff;
  border-color: #20a0ff;
}

.el-table__body,
.el-table__footer,
.el-table__header {
  border: 2px solid red;
}

#account_page {
  padding: 4px 10px 0 20px;
  padding-bottom: 100px;
  background: #fff;
  border: 1px solid #e7ecf1;
  border-top: none;
  min-height: 230px;
}

.el-switch__label,
.el-switch__label * {
  font-size: 12px;
}

ul.roleList li {
  list-style-type: none;
  float: left;
}

div#addaccount_form .el-form-item__label {
  margin-left: 0px !important;
}

span.el-tag {
  /* margin-left: 10px; */
  padding: 0 10px 0 0;
}

i.el-icon-edit,
i.el-icon-close {
  cursor: pointer
}

.eidtRoleBtn {
  width: 120px;
  height: 50px;
}

.eidtRoleBtn:nth-of-type(1):hover {
  background: rgba(248, 126, 43, 0.9);
}

.eidtRoleBtn:nth-of-type(1) {
  background: #f87e2b;
  border: none;
  color: #fff;
}

.eidtRoleBtn:nth-of-type(2) {
  background: #fff;
  color: #444;
  border: 1px solid rgba(196, 196, 196, 1)
}

.eidtRoleBtn:nth-of-type(2):hover {
  border: 1px solid rgb(248, 126, 43);
  color: rgb(248, 126, 43);
}

div.account>h1 .addRoleBtn {
  width: 120px;
  height: 50px;
  font-size: 14px;
}

div.account>h1 .addRoleBtn:nth-of-type(1):hover {
  background: rgba(248, 126, 43, 0.9);
}

div.account>h1 .addRoleBtn:nth-of-type(1) {
  background: #f87e2b;
  border: none;
  color: #fff;
  margin-left: 100px;
}

div.account>h1 .addRoleBtn:nth-of-type(2) {
  background: #fff;
  color: #444;
  border: 1px solid rgba(196, 196, 196, 1)
}

div.account>h1 .addRoleBtn:nth-of-type(2):hover {
  border: 1px solid rgb(248, 126, 43);
  color: rgb(248, 126, 43);
}

button#roleSearchBtn {
  width: 80px;
  /* float: right; */
  height: 36px;
  line-height: 11px;
  margin-right: 20px;
  color: #fff;
  margin-top: 18px;
  outline: none;
  border: none;
  /* border-radius: 4px; */
  background: rgba(52, 52, 67, 0.8);
}

button#roleSearchBtn:hover {
  color: #fff
}

div.editfooter {
  text-align: left;
  padding-left: 210px;
  margin-top: -43px;
}

div.addfooter {
  text-align: left;
  padding-left: 120px;
  margin-top: -51px;
}

div.rolename {
  font-weight: normal;
}
</style>

