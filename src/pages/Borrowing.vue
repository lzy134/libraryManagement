<template>
  <div class="borrowContainer">
    <!-- 面包屑 -->
    <Breadcrumb :breadcrumbList="breadcrumbList"></Breadcrumb>
    <!-- <el-card> -->
    <el-tabs type="border-card" v-model="tabName" style="margin-top:20px">
      <el-tab-pane label="所有借阅" name="first">
        <Eltable :tableArr="tableArr" :getKeyword="getKeyword" :keyword="keyword" :tabName="tabName"></Eltable>
      </el-tab-pane>
      <el-tab-pane label="待还借阅" name="second">
        <Eltable :tableArr="toBePaid" :getKeyword="getKeyword" :keyword="keyword" :tabName="tabName" :getAddBtn="getAddBtn" :returnBookById="returnBookById"></Eltable>
      </el-tab-pane>
      <el-tab-pane label="逾期借阅" name="third">
        <Eltable :tableArr="overdue" :getKeyword="getKeyword" :keyword="keyword" :tabName="tabName"></Eltable>
      </el-tab-pane>
    </el-tabs>
    <el-dialog
      title="添加借阅信息"
      :visible.sync="isAddBorrowDialog"
      width="40%"
      @close="closeDialog"
    >
      <el-form :model="addBorrowData" label-width="80px" ref="borrowRef" :rules="borrowRules">
        <el-form-item label="用户名" prop="account">
          <el-input v-model="addBorrowData.account"></el-input>
        </el-form-item>
        <el-form-item label="图书名" prop="bookName">
          <!-- <el-input v-model="addBorrowData.bookName"></el-input> -->
          <el-cascader
            v-model="addBorrowData.bookName"
            :options="bookOptions"
            @change="handleChange"></el-cascader>
        </el-form-item>
        <el-form-item label="借阅时间" prop="borrowTime">
          <el-date-picker
            v-model="addBorrowData.borrowTime"
            type="date"
            value-format="yyyy-MM-dd"
            placeholder="借阅日期">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="归还时间" prop="planReturnTime">
          <el-date-picker
            v-model="addBorrowData.planReturnTime"
            type="date"
            value-format="yyyy-MM-dd"
            placeholder="归还日期">
          </el-date-picker>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isAddBorrowDialog = false">取 消</el-button>
        <el-button type="primary" @click="submitBorrow('borrowRef')">确 定</el-button>
      </span>
    </el-dialog>  
  </div>
</template>

<script>
  import Breadcrumb from '../components/Breadcrumb'
  import Eltable from '../components/Eltable'
  import {getRequest, postRequest} from '../untils/api'
  import dayjs from 'dayjs'
  export default {
    name: 'Borrowing',
    components: {Breadcrumb, Eltable},
    data() {
      return {
        breadcrumbList: ['您的位置','借阅管理'],
        tabName: 'first',
        tableArr: [],   // 所有借阅信息
        toBePaid: [],   // 待还借阅
        overdue: [],    // 逾期借阅
        keyword: '',
        isAddBorrowDialog: false,  // 控制添加待还借阅对话框显示隐藏
        addBorrowData: {    // 添加待还借阅数据
          account: '',
          bookName: '',
          borrowTime: '',
          planReturnTime: ''
        },
        borrowRules: {
          account: [
            {required: true, message: '请输入用户名', trigger: 'blur'}
          ],
          borrowTime: [
            {required: true, message: '请选择日期', trigger: 'change' }
          ],
          planReturnTime: [
            {required: true, message: '请选择日期', trigger: 'change' }
          ],
          bookName: [
            {required: true, message: '请输入图书名', trigger: 'blur'}
          ],
        },
        bookOptions: []
      }
    },
    mounted() {
      this.getBorrowInfo()
      this.getAllBooks()
    },
    methods: {
      // 获取查询账号的内容
      getKeyword(account) {
        let status = '0'
        if(this.tabName === 'second') {
          status = '1'
        }else if(this.tabName === 'third') {
          status = '2'
        }else {
          status = '0'
        }
        if(!account.trim()) {
          this.getBorrowInfo()
        }else {
          getRequest('/borrow/search', {account,status}).then(resp => {
            if(resp.code !== 0) {
              return this.$message.error(resp.msg)
            }
            this.$message.success(resp.msg)
            resp.data.forEach(item => {
              if(!item.returnTime) {
                item.returnTime = '--'
              }
            })
            if(this.tabName === 'first') {
              this.tableArr = resp.data
            }else if(this.tabName === 'second') {
              this.toBePaid = resp.data
            }else {
              this.overdue = resp.data
            }
          })
        }
        this.keyword = ''
      },
      // 获取所有图书
      getAllBooks() {
        getRequest('/classify').then(res => {
          getRequest('/books').then(resp => {
            res.data.filter(item1 => {
              resp.data.filter(item2 => {
                if(item1.className === item2.classify) {
                  let book = {value: '', label: '', children: []}
                  book.value = item1.className
                  book.label = item1.className
                  book.children.push({value: item2.bookName, label: item2.bookName})
                  this.bookOptions.push(book)
                }
              })
            })
          })
        })
        
        
      },
      // 获取借阅信息
      getBorrowInfo() {
        getRequest('/borrow').then(resp => {
          if(resp.code !== 0) {
            return this.$message.error(resp.msg)
          }
          resp.data.forEach(item => {
            if(!item.returnTime) {
              item.returnTime = '--'
            }
          })
          this.tableArr = resp.data
          this.toBePaid = resp.data.filter(item => item.borrowStatus === 1)
          this.overdue = resp.data.filter(item => item.borrowStatus === 2)
        })
      },
      submitBorrow(borrowRef) {
        this.$refs.borrowRef.validate((valid) => {
          console.log(this.addBorrowData);
          if (valid) {
            postRequest('/borrow/add', this.addBorrowData).then(resp => {
              if(resp.code !== 0) {
                this.$message.error(resp.msg)
                return false
              }
              this.$message.success(resp.msg)
              this.getBorrowInfo()
            })
            this.isAddBorrowDialog = false
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      // 获取用户是否点击了添加按钮
      getAddBtn(val) {
        this.isAddBorrowDialog = val
      },
      // 对话框关闭的回调
      closeDialog() {
        this.$refs.borrowRef.resetFields()
      },
      // 还书
      returnBookById(id) {
        console.log(id);
        const returnTime = dayjs(new Date()).format('YYYY-MM-DD')
        console.log(returnTime);
        postRequest('/borrow/returnBook', {id,returnTime}).then(resp => {
          if(resp.code !== 0) {
            return this.$message.error('还书失败！')
          }
          this.$message.success('还书成功')
          this.getBorrowInfo()
        })
      },
      handleChange() {

      }
    },
  }
</script>

<style lang="less" scoped>
  .borrowContainer {
    width: 100%;
    // height: 100%;
    margin-bottom: 60px;
    .el-card {
      width: 100%;;
      margin-top: 20px;
    }
    
  }
</style>