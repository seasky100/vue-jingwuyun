<!-- 场所类型统计分析 -->
<template>
<div class="echarts-div_box">
  <div class="tactics__box">
    <div class="tactics__box__title">
      {{title}}
    </div>
    <div class="tactics__box__content">
      <el-row :gutter="15">
        <el-col :span="12" style=" height:400px;">
       <img :src="echartImg" v-show="echartImg" />
          <echarts v-show="!echartImg" :option="pieTestOps" style="width:100%; height:100%;" ref="echart"></echarts>
        </el-col>
        <el-col :span="12" style=" height:400px;">
          <div class="tactics__right__title">
            {{title}}
          </div>
          <div class="tactics__right__body">
            <ul class="tactics__right__body__ul">
              <li v-for="(item, key) in zdData" :key='key' class="bodyliw" @click="listhow(key)">
                <el-row :gutter="15">
                  <el-col :span="24">

                    <div class="fx_number">
                      <span class="number">{{key+1 }}</span> {{item.type}}
                      <span>总数：</span>{{item.total}}
                      <!-- <span>中风险：</span>{{item.zfx}}
                      <span>低风险：</span>{{item.dfx}} -->
                    </div>
                  </el-col>
                </el-row>

                <ul class="bodyliw__ul" v-show="listact === key">
                  <li v-for="(items,index) in item.subData" :key="index">
                    <div class="fx_number">
                      <span class="number"></span>{{items.type}}
                      <span>高风险：</span>{{items.gfx}}
                      <span>中风险：</span>{{items.zfx}}
                      <span>低风险：</span>{{items.dfx}}
                    </div>
                  </li>

                </ul>

              </li>
            </ul>
          </div>
        </el-col>
      </el-row>
    </div>
  </div>
</div>
</template>
<style lang="scss" scoped>
  .tactics {
    &__right {
      &__title {
        border-bottom: 2px solid #ccc;
        color: #000;
        padding: 10px 0;
      }
      &__body {
        &__ul {
          list-style: none;
          padding: 0;
          margin: 0;
          height: 295px;
          overflow: auto;
          li.bodyliw {
            position: relative;
            padding-left: 25px;
            font-size: 16px;
            padding: 10px;
            &:before {
              content: '';
              display: block;
              width: 10px;
              height: 10px;
              position: absolute;
              top: 15px;
              left: 10px;
              background: #2078d9;
            }
            &:first-child {
              .number {
                color: #c00;
              }
            }
            &:nth-child(2) {
              .number {
                color: #F33A3A;
              }
            }
            &:nth-child(3) {
              .number {
                color: #F15C5C;
              }
            }
            .number {
              color: #666;
              display: inline-block;
              vertical-align: top;
              margin: 0 10px;
            }
            .fx_number {
              color: #333;
              font-size: 12px;
              padding-left: 10px;
              span {
                display: inline-block;
                vertical-align: top;
                margin: 0 0 0 15px;
              }
            }
            .last_number {
              font-size: 14px;
              color: #85c0e9;
            }
            &:hover {
              background: #eee;
              cursor: pointer;
            }
            .bodyliw__ul {
              margin: 0;
              padding-left: 25px;
              li {
                list-style: none;
                color: #ef5843;
                background-color: #fff4f3;
                margin: 5px;
                .fx_number {
                  color: #ef5843;
                }
              }
            }
          }
        }
      }
    }
  }

</style>

<script>
  import mixin from '../../../../utils/mixin'
  import { clone } from '../../../../utils/util'
  import echarts from '../../../../widgets/echarts'
  import chartsOps from '../../../../utils/echartsOptions/all'
  import mapData from '../../../../utils/echartsOptions/mapData'
  export default {
    mixins: [mixin, chartsOps, mapData],
    components: {
      echarts
    },
    props: ['title', 'chartData'],
    data() {
      return {
        echartImg: '',
        listact: '',
        pieTestOps: null,
        lineTestOps: null,
        mapTestOps: null,
        barTestOps: null,
        list: [],
        zdData: []
      }
    },
    computed: {},
    watch: {
      chartData(val) {
        this.zdData = this.chartData[7].place
        // 饼图数据
        this.pieTestOps = clone(this.echartsPieOptions.full)
        // 涉稳人员、涉恐人员、精神病人、前科人员、涉枪涉爆人员、涉案人员、重点上访人员、吸毒人员
        this.pieTestOps.legend.data = ['重点群体人员', '重点行业单位', '重点场所部位', '队伍和内部管理', '无人驾驶航空器管理', '重点线索管理', '重点案事件']
        this.pieTestOps.series[0].data = [
          { name: '怀化市', value: this.chartData[3].area.hhs },
          { name: '麻阳', value: this.chartData[3].area.mymzzzx },
          { name: '会同', value: this.chartData[3].area.htx },
          { name: '洪江', value: this.chartData[3].area.hjs },
          { name: '鹤城', value: this.chartData[3].area.hcq },
          { name: '辰溪', value: this.chartData[3].area.cxx },
          { name: '芷江', value: this.chartData[3].area.zjdzzzx },
          { name: '沅陵', value: this.chartData[3].area.ylx },
          { name: '溆浦', value: this.chartData[3].area.xpx },
          { name: '新晃', value: this.chartData[3].area.xhdzzzx },
          { name: '靖州', value: this.chartData[3].area.jzmzdzzzx },
          { name: '通道', value: this.chartData[3].area.tddzzzx },
          { name: '中方', value: this.chartData[3].area.zfx }
        ]
        this.pieTestOps.series[0].center = ['50%', '50%']
        this.pieTestOps.series[0].labelLine = {
          show: true
        }
        this.pieTestOps.series[0].label.normal = {
          position: 'outside',
          formatter: '{b}{c}'
        }
      }
    },
    methods: {
      listhow(key) {
        if (this.listact === key) {
          this.listact = ''
        } else {
          this.listact = key
        }
      }
    },
    created() {},
    mounted() {}
  }

</script>
