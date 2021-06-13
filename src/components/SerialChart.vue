<template>
  <div style="max-width: 800px;margin: 0 auto">
    <el-row>
      <el-col :span="8">
        <el-select :disabled="serialIsConn" style="width: 100%; height: 100%;" v-model="serialBaudRate"
                   placeholder="波特率">
          <el-option
              v-for="item in serialBaudOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="8">
        <el-button @click="openSerial" :disabled="serialBaudRate == ''" style="width: 100%; height: 100%;">连接串口
        </el-button>
      </el-col>
      <el-col :span="8">
        <el-button @click="cleanChart" :disabled="serialIsConn" style="width: 100%; height: 100%;">清空记录</el-button>
      </el-col>
    </el-row>
    <!--    <el-row>-->
    <!--      <el-col :span="8">-->
    <!--        <el-button @click="addData" style="width: 100%; height: 100%;">add data</el-button>-->
    <!--      </el-col>-->
    <!--      <el-col :span="8">-->
    <!--        <el-button @click="addDataset" style="width: 100%; height: 100%;">add dataset</el-button>-->
    <!--      </el-col>-->
    <!--      <el-col :span="8">-->
    <!--        <el-button @click="removeDataset" style="width: 100%; height: 100%;">remove dataset</el-button>-->
    <!--      </el-col>-->
    <!--    </el-row>-->
    <canvas ref="chart" width="800px" height="600px"></canvas>
  </div>
</template>

<script>
import {Chart, registerables} from 'chart.js'
import {valueOrDefault} from 'chart.js/helpers'
import colorLib from '@kurkle/color'

const COLORS = [
  '#4dc9f6',
  '#f67019',
  '#f53794',
  '#acc236',
  '#537bc4',
  '#166a8f',
  '#00a950',
  '#58595b',
  '#8549ba'
]

export default {
  name: "SerialChart",
  data() {
    return {
      showNum: 50,
      serialBaudOptions: [{
        value: 9600,
        label: '9600'
      }, {
        value: 115200,
        label: '115200'
      }, {
        value: 921600,
        label: '921600'
      }],
      serialBaudRate: '',
      serialChart: null,
      serialChartConfig: {
        type: 'line',
        data: {
          labels: [],
          datasets: [{}],
        }
      },
      serialIsConn: false,
      count: 0,
    }
  },
  mounted() {
    this.initChart()
    setInterval(() => {
      let array = []
      array.push(this.rand(0,100))
      array.push(this.rand(0,100))
      array.push(this.rand(0,100))
      array.push(this.rand(0,100))
      // console.log(array)
      this.addDataArray(array)
    }, 100)
    // console.error(this.serialChart)
  },
  methods: {
    initChart() {
      //初始化位置
      for (let i = 0; i < this.showNum; ++i) {
        this.serialChartConfig.data.labels.push(i)
      }
      if (this.serialChart == null) {
        let ctx = this.$refs.chart.getContext('2d')
        Chart.register(...registerables)
        this.serialChart = new Chart(ctx, this.serialChartConfig)
      }
      this.serialChart.data.datasets.pop()
      this.serialChart.update()
    },
    // addData() {
    //   this.count++
    //   const data = this.serialChart.data
    //   if (data.datasets.length > 0) {
    //     data.labels.shift()
    //     data.labels.push(`${data.labels.length + this.count}`)
    //     for (let index = 0; index < data.datasets.length; ++index) {
    //       data.datasets[index].data.shift()
    //       data.datasets[index].data.push(this.rand(0, 100))
    //     }
    //     this.serialChart.update()
    //   }
    // },
    // addDataset() {
    //   const data = this.serialChart.data
    //   const dsColor = this.namedColor(this.serialChart.data.datasets.length)
    //   const newDataset = {
    //     label: '数据 ' + (data.datasets.length + 1),
    //     backgroundColor: this.transparentize(dsColor, 0.5),
    //     borderColor: dsColor,
    //     data: this.numbers({count: data.labels.length, min: 0, max: 100}),
    //   }
    //   this.serialChart.data.datasets.push(newDataset)
    //   this.serialChart.update()
    // },
    // removeDataset() {
    //   this.serialChart.data.datasets.pop()
    //   this.serialChart.update()
    // },
    // 清除表格
    cleanChart() {
      this.count = 0
      this.serialChart.data.datasets = []
      this.serialChart.update()
    },
    // 根据数据长度，修改
    checkData(array) {
      for (let index = 0; index < array.length; index++) {
        const dsColor = this.namedColor(index)
        const newDataset = {
          label: '数据 ' + (index + 1),
          backgroundColor: this.transparentize(dsColor, 0.5),
          borderColor: dsColor,
          data: [],
        }
        this.serialChart.data.datasets.push(newDataset)
      }
      this.serialChart.update()
    },
    addDataArray(array) {
      // 还没有收到第一帧数据时，初始化表格dataset结构
      if (array.length !== 0 && this.serialChart.data.datasets.length === 0) {
        this.checkData(array)
      }
      if (this.count > (this.showNum - 1)) {
        this.serialChart.data.labels.push(this.count)
        //删除第一个X
        this.serialChart.data.labels.shift()
        //删除所有数据第一个Y
        for (let index = 0; index < array.length; index++) {
          this.serialChart.data.datasets[index].data.shift()
        }
      }
      array.forEach((value, index) => {
        this.serialChart.data.datasets[index].data.push(value)
      })
      this.count++
      this.serialChart.update()
    },
    /**
     * serial
     */
    async openSerial() {
      try {
        let port = await navigator.serial.requestPort({})
        await port.open({baudrate: this.serialBaudOptions})
      } catch (e) {
        console.error(e)
        this.$message.warning("连接失败")
      }
    },
    /**
     * utils
     */
    namedColor(index) {
      return COLORS[index % COLORS.length]
    },
    transparentize(value, opacity) {
      let alpha = opacity === undefined ? 0.5 : 1 - opacity
      return colorLib(value).alpha(alpha).rgbString()
    },
    numbers(config) {
      let cfg = config || {}
      let min = valueOrDefault(cfg.min, 0)
      let max = valueOrDefault(cfg.max, 100)
      let count = valueOrDefault(cfg.count, 8)
      let data = []
      let i
      for (i = 0; i < count; ++i) {
        data.push(this.rand(min, max))
      }
      return data
    },
    rand(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min
    }
  }
}
</script>

<style scoped>

</style>