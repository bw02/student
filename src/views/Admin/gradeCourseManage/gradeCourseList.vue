<template>
  <div>
    <div id="pie" style="width: 600px; height: 400px;"></div>
    <el-table
        :data="tableData"
        border
        stripe
        style="width: 100%">
      <el-table-column
          fixed
          prop="cid"
          label="课程号"
          width="150">
      </el-table-column>
      <el-table-column
          prop="cname"
          label="课程名"
          width="150">
      </el-table-column>
      <el-table-column
          fixed
          prop="tid"
          label="工号"
          width="100">
      </el-table-column>
      <el-table-column
          prop="tname"
          label="教师名"
          width="100">
      </el-table-column>
      <el-table-column
          fixed
          prop="sid"
          label="学号"
          width="100">
      </el-table-column>
      <el-table-column
          prop="sname"
          label="学生名"
          width="100">
      </el-table-column>
      <el-table-column
          prop="grade"
          label="成绩"
          width="100">
      </el-table-column>
      <el-table-column
          prop="term"
          label="学期"
          width="100">
      </el-table-column>
      <el-table-column
          label="操作"
          width="100">
        <template slot-scope="scope">
          <el-popconfirm
              confirm-button-text='删除'
              cancel-button-text='取消'
              icon="el-icon-info"
              icon-color="red"
              title="删除不可复原"
              @confirm="deleteTeacher(scope.row)"
          >
            <el-button slot="reference" type="text" size="small">删除</el-button>
          </el-popconfirm>
          <el-button @click="editor(scope.row)" type="text" size="small">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
        background
        layout="prev, pager, next"
        :total="total"
        :page-size="pageSize"
        @current-change="changePage"
    >
    </el-pagination>
  </div>
  
</template>

<script>
import * as echarts from 'echarts'
export default {
  methods: {
    select(row) {
      console.log(row)
    },
    deleteTeacher(row) {
      const that = this
      console.log(row)
      const sid = row.sid
      const cid = row.cid
      const tid = row.tid
      const term = row.term
      axios.get("http://localhost:10086/SCT/deleteById/" + sid + '/' + cid + '/' + tid + '/' + term).then(function (resp) {
        console.log(resp)
        if (resp.data === true) {
          that.$message({
            showClose: true,
            message: '删除成功',
            type: 'success'
          });
          window.location.reload()
        }
        else {
          that.$message({
            showClose: true,
            message: '删除出错，请查询数据库连接',
            type: 'error'
          });
        }
      }).catch(function (error) {
        that.$message({
          showClose: true,
          message: '删除出错，存在外键依赖',
          type: 'error'
        });
      })
    },
    changePage(page) {
      page = page - 1
      const that = this
      let start = page * that.pageSize, end = that.pageSize * (page + 1)
      let length = that.tmpList.length
      let ans = (end < length) ? end : length
      that.tableData = that.tmpList.slice(start, ans)
    },
    editor(row) {
      this.$router.push({
        path: '/editorGradeCourse',
        query: {
          cid: row.cid,
          tid: row.tid,
          sid: row.sid,
          term: row.term
        }
      })
    }, 
    calculateGradeDistribution() {
    const ranges = {
      '90分及以上': 0,
      '80到89分': 0,
      '70到79分': 0,
      '60到69分': 0,
      '60分以下': 0
    };

    this.tmpList.forEach(student => {
      const grade = student.grade;
      if (grade === null || grade === undefined) {
          return; 
      }
      if (grade >= 90) {
        ranges['90分及以上']++;
      } else if (grade >= 80) {
        ranges['80到89分']++;
      } else if (grade >= 70) {
        ranges['70到79分']++;
      } else if (grade >= 60) {
        ranges['60到69分']++;
      } else {
        ranges['60分以下']++;
      }
    });

    this.gradeDistribution = Object.keys(ranges).map(key => ({ name: key, value: ranges[key] }));
    this.drawPieChart();//绘制饼图
  },
  drawPieChart() {
    const pie = document.getElementById('pie');
    if (!pie) return;
    const pieChart = echarts.init(pie);
    const option = {
      title: {
        text: '当前学期学生成绩分布',
        left: 'center'
      },
      tooltip: {
        trigger: 'item'
      },
      legend: {
        orient: 'vertical',
        left: 'left',
      },
      series: [
        {
          name: '成绩分布',
          type: 'pie',
          radius: '50%',
          data: this.gradeDistribution
        }
      ]
    };
    pieChart.setOption(option);
  }
  },
  data() {
    return {
      tableData: null,
      pageSize: 10,
      total: null,
      tmpList: null,
      gradeDistribution: [], // 用于存储分数区间和学生人数的数据
    }
  },
  props: {
    ruleForm: Object,
  },
  watch: {
    ruleForm: {
      handler(newRuleForm, oldRuleForm) {
        console.log("组件监听 form")
        console.log(newRuleForm)
        const that = this
        that.tmpList = null
        that.total = null
        that.tableData = null
        axios.post("http://localhost:10086/SCT/findBySearch", newRuleForm).then(function (resp) {
          console.log("查询结果:");
          console.log(resp)
          that.tmpList = resp.data
          that.total = resp.data.length
          let start = 0, end = that.pageSize
          let length = that.tmpList.length
          let ans = (end < length) ? end : length
          that.tableData = that.tmpList.slice(start, ans)
          that.calculateGradeDistribution();  // 计算分数区间
        })
      },
      deep: true,
      immediate: true
    }
  },
}
</script>