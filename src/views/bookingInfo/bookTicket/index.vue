<template>
  <div class="activityInfo">
    <!-- 表格顶部 -->
    <div class="header">
      <el-date-picker v-model="datePicker" type="date" size="small" placeholder="选择日期">
      </el-date-picker>
      <el-button type="primary" size="mini" @click='goSearch'>查询</el-button>
      <el-button type="primary" size="small" @click="$router.push('/home/bookTicket/edit')">添加门票套餐</el-button>
    </div>
    <!-- 表格 -->
    <el-table :data="buyData" stripe size="small" :default-sort="{prop: 'date', order: 'descending'}"
      :cell-style="{padding:'1px 0'}" :header-cell-style="{background:'#ddd'}">
      <el-table-column prop="num" label="序号" sortable width="80">
      </el-table-column>
      <el-table-column prop="id" label="订单号" sortable width="280">
      </el-table-column>
      <el-table-column prop="username" label="姓名" width="200">
      </el-table-column>
      <el-table-column label="门票日期" width="150">
        <template slot-scope="scope">
        
            <div slot="reference" class="name-wrapper">
              <el-tag size="medium">{{ scope.row.title }}</el-tag>
            </div>
        </template>
      </el-table-column>
      <el-table-column prop="number" label="游玩人数" width="80">
      </el-table-column>
      <el-table-column prop="price" label="购票价格" width="120">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页符 -->
    <el-pagination class="pagination" background @size-change="handleSizeChange" @current-change="handleCurrentChange"
      :current-page="currentPage" :page-size="10" layout="total, prev, pager, next, jumper" :total="buyData.length">
    </el-pagination>
    </el-pagination>
  </div>
</template>

<script>
import {
  buySearch,
  findTicketById,
  deleteTicket

} from '@/utils/apply.url';
export default {
  name: 'activityInfo',
  data () {
    return {
      currentPage: 1,
      buyData: [],
      datePicker: ''
    }
  },
  mounted () {
    this.datePicker = new Date()
    this.getBuyData();
  },
  methods: {
    timestampToTime (timestamp) {
      var date = new Date(timestamp);
      var Y = date.getFullYear() + '-';
      var M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-';
      var D = (date.getDate() < 10 ? '0' + date.getDate() : date.getDate());
      return Y + M + D;
    },
    // 获取购票数据
    getBuyData () {
      this.buyData = [];
      buySearch({
        selectPlayDate: this.timestampToTime(this.datePicker)
      }, 'get').then(res => {
        if (res.length != 0) {
          var index = 1;
          var obj = {};
          for (let i = 0; i < res.length; i++) {
            obj = {
              username: res[i].idName,
              selectPlayDate: res[i].selectPlayDate,
              num: index++,
              id: res[i].id,
              number: res[i].number,
              price: res[i].ticketPay,
              ticketId: res[i].ticketId,
              title: [],
              desc: []
            };
            this.buyData.push(obj)
          }
          this.getTicketDetail();
        } else {
          this.$message.success(res.msg);
        }
      }).catch(err => {
        this.$message.error('获取失败');
      });
    },
    getTicketDetail () {
      this.buyData.forEach((item, index) => {
        findTicketById({
          ticketId: item.ticketId
        }, 'get').then(result => {
          console.log(111)
          this.buyData[index].title = result[0].title,
            this.buyData[index].desc = result[0].descs;
          console.log(this.buyData[index])
        }).catch(error => {
          console.log(222)
          this.$message.error('获取电子门票信息失败');
        })
      })
    },
    // 搜索
    goSearch () {
      this.getBuyData();
    },
    // 编辑活动资讯
    handleEdit (index, row) {
      console.log(row);
      this.$router.push({
        path: '/home/activityInfo/edit',
        query: {
          id: row.id,
          title: row.title,
          desc: row.desc
        }
      });
    },
    handleDelete (index, row) {
      console.log(index, row.id);
      deleteTicket({
        id: row.id
      }, 'post').then(res => {
        this.$message.success(res.msg);
      }).catch(err => {
        console.log(22)
        this.$message.success('删除失败' || res.msg);
      });
      this.getBuyData();
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`);
      this.getBuyData(val);
    },
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`);
    },
  }
}

</script>

<style rel="stylesheet/scss" lang="scss" scope>
.header {
  margin-bottom: 10px;

  .el-input {
    width: 200px;
  }

  .el-button--mini {
    /* margin-left: 10px; */
  }

  .el-button--small {
    float: right;
  }
}

.activity-info .el-tag--medium {
  width: 200px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  vertical-align: middle;
  margin: 4px 0;
}

.el-popover {
  width: 400px;
}

.pagination {
  float: right;
  margin-top: 10px;
}
</style>
