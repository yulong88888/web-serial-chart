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
        <el-button @click="doSerial" :disabled="serialBaudRate === ''"
                   style="width: 100%; height: 100%;">{{ serialIsConn ? '断开串口' : '连接串口' }}
        </el-button>
      </el-col>
      <el-col :span="8">
        <el-button @click="cleanChart" style="width: 100%; height: 100%;">清空记录</el-button>
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
      serialIsConn: false,
      serialPort: null,
      serialPortReader: null,
      serialChart: null,
      serialChartConfig: {
        type: 'line',
        data: {
          labels: [],
          datasets: [{}],
        }
      },
      count: 0,
    }
  },
  mounted() {
    this.initChart()
    // setInterval(() => {
    //   let array = []
    //   array.push(this.rand(0,100))
    //   array.push(this.rand(0,100))
    //   array.push(this.rand(0,100))
    //   array.push(this.rand(0,100))
    //   // console.log(array)
    //   this.addDataArray(array)
    // }, 100)

    // navigator.serial.addEventListener('connect', (event) => {
    //   console.log(event)
    //   this.$message.success("连接成功")
    // })

    navigator.serial.addEventListener('disconnect', (event) => {
      console.log(event)
      this.closeSerial()
      this.$message.error("连接断开")
    })
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
      this.serialChartConfig.data.labels = []
      this.serialChart.data.datasets = []
      for (let i = 0; i < this.showNum; ++i) {
        this.serialChartConfig.data.labels.push(i)
      }
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
    doSerial() {
      if (this.serialIsConn) {
        this.closeSerial()
      } else {
        this.openSerial()
      }
    },
    async openSerial() {
      try {
        this.serialPort = await navigator.serial.requestPort({})
        console.log(this.serialPort)
        await this.serialPort.open({baudRate: this.serialBaudRate})

        this.serialIsConn = true

        let lineBuffer = ''
        this.serialPortReader = this.serialPort.readable.getReader()
        let decoder = new TextDecoder()

        while (this.serialPort && this.serialPort.readable) {
          try {
            for (; ;) {
              const {value, done} = await this.serialPortReader.read()
              if (value) {
                let str = decoder.decode(value)
                // console.log(str)
                lineBuffer += str
                let lines = lineBuffer.split('\n')
                if (lines.length > 1) {
                  // console.error(lineBuffer)
                  let arrayStr = lineBuffer.split(",")
                  // console.log(arrayStr)
                  let data = arrayStr.map(item => {
                    return Number(item)
                  })
                  // console.error(data)
                  this.addDataArray(data)
                  lineBuffer = ""
                }
              }
              if (done) {
                console.error(done)
                break
              }
            }
            this.serialPortReader.releaseLock()
            this.serialPortReader = null
          } catch (e) {
            console.error(e)
            this.serialIsConn = false
          }
        }
      } catch (e) {
        console.error(e)
        this.serialIsConn = false
        this.$message.warning("连接失败")
      }
    },
    async closeSerial() {
      const localPort = this.serialPort
      this.serialPort = null

      if (this.serialPortReader) {
        await this.serialPortReader.cancel()
      }

      if (localPort) {
        try {
          await localPort.close()
        } catch (e) {
          console.error(e)
        }
      }

      this.serialPortReader = null
      this.serialPort = null
      this.serialIsConn = false
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
    // numbers(config) {
    //   let cfg = config || {}
    //   let min = valueOrDefault(cfg.min, 0)
    //   let max = valueOrDefault(cfg.max, 100)
    //   let count = valueOrDefault(cfg.count, 8)
    //   let data = []
    //   let i
    //   for (i = 0; i < count; ++i) {
    //     data.push(this.rand(min, max))
    //   }
    //   return data
    // },
    // rand(min, max) {
    //   return Math.floor(Math.random() * (max - min + 1)) + min
    // }
  }
}
</script>

<style scoped>

</style>
