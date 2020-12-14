<template>
  <div>
    <!--    导航区-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
      <div id="main" class="charts" style="width: 600px;height:400px;"></div>
    </el-card>
  </div>
</template>

<script>
import echarts from 'echarts'
export default {
  data () {
    return {
    }
  },
  created() {
  },
  // 执行到mounted函数此时页面上元素已经被渲染完毕了
  async mounted() {
    const dom = document.getElementById('main')
    if (dom) {
      echarts.init(dom)
    } else {
      alert('dom不存在')
    }
    // console.log(myChart)
    // const { data: res } = await this.$http.get('reports/type/1')
    // if (res.meta.status !== 200) {
    //   return this.$message.error(res.meta.msg)
    // }
    // myChart.setOption(res.data)
  },
  updated() {
  },
  methods: {
  }
}
</script>

<style scoped>

</style>
