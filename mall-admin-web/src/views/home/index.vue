<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { str2Date } from '@/utils/datetime'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { LineChart } from 'echarts/charts'
import VChart from 'vue-echarts'
import {
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent
} from 'echarts/components'

use([
  CanvasRenderer,
  LineChart,
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent
])

type LineChartDataItem = {
  date: string
  orderCount: number
  orderAmount: number
}

const defaultLineChartData: LineChartDataItem[] = [
  { date: '2026-01-01', orderCount: 10, orderAmount: 1093 },
  { date: '2026-01-02', orderCount: 20, orderAmount: 2230 },
  { date: '2026-01-03', orderCount: 33, orderAmount: 3623 },
  { date: '2026-01-04', orderCount: 50, orderAmount: 6423 },
  { date: '2026-01-05', orderCount: 80, orderAmount: 8492 },
  { date: '2026-01-06', orderCount: 60, orderAmount: 6293 },
  { date: '2026-01-07', orderCount: 20, orderAmount: 2293 },
  { date: '2026-01-08', orderCount: 60, orderAmount: 6293 },
  { date: '2026-01-09', orderCount: 50, orderAmount: 5293 },
  { date: '2026-01-10', orderCount: 30, orderAmount: 3293 },
  { date: '2026-01-11', orderCount: 20, orderAmount: 2293 },
  { date: '2026-01-12', orderCount: 80, orderAmount: 8293 },
  { date: '2026-01-13', orderCount: 100, orderAmount: 10293 },
  { date: '2026-01-14', orderCount: 10, orderAmount: 1293 },
  { date: '2026-01-15', orderCount: 40, orderAmount: 4293 }
]

const defaultStartDate = new Date(2026, 0, 1)

const datePickerRange = ref<Date[]>([])
const lineChartData = ref<LineChartDataItem[]>([])
const loading = ref(false)

const initDatePickerRange = () => {
  const start = defaultStartDate
  const end = new Date(start.getTime() + 1000 * 60 * 60 * 24 * 7)
  datePickerRange.value = [start, end] as Date[]
}

const getLineChartData = () => {
  loading.value = true
  setTimeout(() => {
    const start = datePickerRange.value[0]
    const end = datePickerRange.value[1]
    lineChartData.value = defaultLineChartData.filter(item => {
      const currDate = str2Date(item.date)
      return currDate!.getTime() >= start!.getTime() && currDate!.getTime() <= end!.getTime()
    })
    loading.value = false
  }, 800)
}

onMounted(() => {
  initDatePickerRange()
  getLineChartData()
})

const shortcuts = [
  {
    text: '最近一周',
    value: () => {
      const start = defaultStartDate
      const end = new Date(start.getTime() + 1000 * 60 * 60 * 24 * 7)
      return [start, end]
    }
  },
  {
    text: '最近一月',
    value: () => {
      const start = defaultStartDate
      const end = new Date(start.getTime() + 1000 * 60 * 60 * 24 * 30)
      return [start, end]
    }
  }
]

const handleDatePickerRangeChange = () => {
  getLineChartData()
}

const chartOption = computed(() => {
  const dates = lineChartData.value.map(item => item.date)
  const orderCounts = lineChartData.value.map(item => item.orderCount)
  const orderAmounts = lineChartData.value.map(item => item.orderAmount)
  return {
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'cross' },
      backgroundColor: '#fff',
      borderColor: '#E2E8F0',
      textStyle: { color: '#1E293B' },
      boxShadow: '0 4px 12px rgba(0,0,0,0.08)'
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      top: 8,
      containLabel: true
    },
    xAxis: {
      type: 'category',
      boundaryGap: false,
      data: dates,
      axisLine: { lineStyle: { color: '#E2E8F0' } },
      axisLabel: { color: '#94A3B8' },
      axisTick: { show: false }
    },
    yAxis: [
      {
        type: 'value',
        name: '数量',
        position: 'left',
        axisLabel: { color: '#94A3B8' },
        splitLine: { lineStyle: { color: '#F1F5F9' } },
        axisLine: { show: false },
        axisTick: { show: false }
      },
      {
        type: 'value',
        name: '金额',
        position: 'right',
        axisLabel: { color: '#94A3B8' },
        splitLine: { show: false },
        axisLine: { show: false },
        axisTick: { show: false }
      }
    ],
    series: [
      {
        name: '订单数量',
        type: 'line',
        smooth: true,
        symbol: 'none',
        areaStyle: {
          color: {
            type: 'linear',
            x: 0, y: 0, x2: 0, y2: 1,
            colorStops: [
              { offset: 0, color: 'rgba(79, 70, 229, 0.12)' },
              { offset: 1, color: 'rgba(79, 70, 229, 0)' }
            ]
          }
        },
        lineStyle: { color: '#4F46E5', width: 2.5 },
        itemStyle: { color: '#4F46E5' },
        data: orderCounts
      },
      {
        name: '订单金额',
        type: 'line',
        yAxisIndex: 1,
        smooth: true,
        symbol: 'none',
        areaStyle: {
          color: {
            type: 'linear',
            x: 0, y: 0, x2: 0, y2: 1,
            colorStops: [
              { offset: 0, color: 'rgba(16, 185, 129, 0.12)' },
              { offset: 1, color: 'rgba(16, 185, 129, 0)' }
            ]
          }
        },
        lineStyle: { color: '#10B981', width: 2.5 },
        itemStyle: { color: '#10B981' },
        data: orderAmounts
      }
    ]
  }
})
</script>

<template>
  <div class="dashboard">
    <!-- Stat Cards -->
    <el-row :gutter="20" class="stat-row">
      <el-col :span="8">
        <div class="stat-card">
          <div class="stat-card-icon stat-card-icon--blue">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4"/>
            </svg>
          </div>
          <div class="stat-card-body">
            <div class="stat-card-label">今日订单总数</div>
            <div class="stat-card-value">200</div>
            <div class="stat-card-trend trend-up">+12% 较昨日</div>
          </div>
        </div>
      </el-col>
      <el-col :span="8">
        <div class="stat-card">
          <div class="stat-card-icon stat-card-icon--green">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
            </svg>
          </div>
          <div class="stat-card-body">
            <div class="stat-card-label">今日销售总额</div>
            <div class="stat-card-value">&yen;5,000</div>
            <div class="stat-card-trend trend-up">+8% 较昨日</div>
          </div>
        </div>
      </el-col>
      <el-col :span="8">
        <div class="stat-card">
          <div class="stat-card-icon stat-card-icon--purple">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M11 3.055A9.001 9.001 0 1020.945 13H11V3.055z"/>
              <path d="M20.488 9H15V3.512A9.025 9.025 0 0120.488 9z"/>
            </svg>
          </div>
          <div class="stat-card-body">
            <div class="stat-card-label">昨日销售总额</div>
            <div class="stat-card-value">&yen;4,620</div>
            <div class="stat-card-trend trend-down">-5% 较前日</div>
          </div>
        </div>
      </el-col>
    </el-row>

    <!-- Chart + Stats Sidebar -->
    <el-row :gutter="20" class="chart-row">
      <el-col :span="17">
        <div class="panel">
          <div class="panel-header">
            <span class="panel-title">订单统计</span>
            <el-date-picker
              size="small"
              v-model="datePickerRange"
              type="daterange"
              align="right"
              unlink-panels
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              :shortcuts="shortcuts"
              @change="handleDatePickerRangeChange"
            />
          </div>
          <div class="panel-body chart-body">
            <v-chart v-if="!loading" :option="chartOption" autoresize class="chart" />
            <div v-else class="chart-loading">
              <el-skeleton :rows="5" animated />
            </div>
          </div>
        </div>
      </el-col>
      <el-col :span="7">
        <div class="panel">
          <div class="panel-header">
            <span class="panel-title">订单概要</span>
          </div>
          <div class="panel-body">
            <div class="summary-item">
              <div class="summary-label">本月订单总数</div>
              <div class="summary-value">10,000</div>
              <div class="summary-trend">
                <span class="trend-up">+10%</span>
                <span class="summary-compare">同比上月</span>
              </div>
            </div>
            <div class="summary-item">
              <div class="summary-label">本周订单总数</div>
              <div class="summary-value">1,000</div>
              <div class="summary-trend">
                <span class="trend-down">-10%</span>
                <span class="summary-compare">同比上周</span>
              </div>
            </div>
            <div class="summary-item">
              <div class="summary-label">本月销售总额</div>
              <div class="summary-value">&yen;100,000</div>
              <div class="summary-trend">
                <span class="trend-up">+10%</span>
                <span class="summary-compare">同比上月</span>
              </div>
            </div>
            <div class="summary-item">
              <div class="summary-label">本周销售总额</div>
              <div class="summary-value">&yen;50,000</div>
              <div class="summary-trend">
                <span class="trend-down">-10%</span>
                <span class="summary-compare">同比上周</span>
              </div>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>

    <!-- Bottom Sections -->
    <el-row :gutter="20" class="bottom-row">
      <!-- Pending Items -->
      <el-col :span="12">
        <div class="panel">
          <div class="panel-header">
            <span class="panel-title">待处理事务</span>
          </div>
          <div class="panel-body panel-body--flush">
            <div class="todo-grid">
              <div class="todo-item">
                <span class="todo-label">待付款订单</span>
                <el-tag size="small" type="danger" round>10</el-tag>
              </div>
              <div class="todo-item">
                <span class="todo-label">待发货订单</span>
                <el-tag size="small" type="warning" round>10</el-tag>
              </div>
              <div class="todo-item">
                <span class="todo-label">已完成订单</span>
                <el-tag size="small" type="success" round>10</el-tag>
              </div>
              <div class="todo-item">
                <span class="todo-label">待确认收货</span>
                <el-tag size="small" round>10</el-tag>
              </div>
              <div class="todo-item">
                <span class="todo-label">已发货订单</span>
                <el-tag size="small" type="primary" round>10</el-tag>
              </div>
              <div class="todo-item">
                <span class="todo-label">待处理退款</span>
                <el-tag size="small" type="danger" round>10</el-tag>
              </div>
              <div class="todo-item">
                <span class="todo-label">新缺货登记</span>
                <el-tag size="small" type="warning" round>10</el-tag>
              </div>
              <div class="todo-item">
                <span class="todo-label">待处理退货</span>
                <el-tag size="small" round>10</el-tag>
              </div>
              <div class="todo-item">
                <span class="todo-label">广告位到期</span>
                <el-tag size="small" type="info" round>10</el-tag>
              </div>
            </div>
          </div>
        </div>
      </el-col>

      <!-- Overview -->
      <el-col :span="6">
        <div class="panel">
          <div class="panel-header">
            <span class="panel-title">商品总览</span>
          </div>
          <div class="panel-body">
            <div class="overview-grid">
              <div class="overview-item">
                <div class="overview-value color-danger">100</div>
                <div class="overview-label">已下架</div>
              </div>
              <div class="overview-item">
                <div class="overview-value color-success">400</div>
                <div class="overview-label">已上架</div>
              </div>
              <div class="overview-item">
                <div class="overview-value color-warning">50</div>
                <div class="overview-label">库存紧张</div>
              </div>
              <div class="overview-item">
                <div class="overview-value color-main">500</div>
                <div class="overview-label">全部商品</div>
              </div>
            </div>
          </div>
        </div>
      </el-col>

      <el-col :span="6">
        <div class="panel">
          <div class="panel-header">
            <span class="panel-title">用户总览</span>
          </div>
          <div class="panel-body">
            <div class="overview-grid">
              <div class="overview-item">
                <div class="overview-value color-main">100</div>
                <div class="overview-label">今日新增</div>
              </div>
              <div class="overview-item">
                <div class="overview-value color-main">200</div>
                <div class="overview-label">昨日新增</div>
              </div>
              <div class="overview-item">
                <div class="overview-value color-success">1,000</div>
                <div class="overview-label">本月新增</div>
              </div>
              <div class="overview-item">
                <div class="overview-value color-main">5,000</div>
                <div class="overview-label">会员总数</div>
              </div>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<style lang="scss" scoped>
@use '@/styles/variables.scss' as v;

.dashboard {
  padding: 4px 0;
}

// ============================================================
// Stat Cards
// ============================================================
.stat-row {
  margin-bottom: 20px;
}

.stat-card {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 24px;
  background: v.$bg-card;
  border-radius: v.$border-radius-lg;
  box-shadow: v.$shadow-sm;
  transition: all v.$transition-normal;
  cursor: default;

  &:hover {
    box-shadow: v.$shadow-md;
    transform: translateY(-2px);
  }
}

.stat-card-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 52px;
  height: 52px;
  min-width: 52px;
  border-radius: 12px;

  svg {
    width: 26px;
    height: 26px;
  }

  &--blue {
    background: #EEF2FF;
    color: var(--el-color-primary);
  }

  &--green {
    background: #ECFDF5;
    color: var(--el-color-success);
  }

  &--purple {
    background: #F5F3FF;
    color: #7C3AED;
  }
}

.stat-card-body {
  min-width: 0;
}

.stat-card-label {
  font-size: 13px;
  color: v.$text-secondary;
  margin-bottom: 4px;
}

.stat-card-value {
  font-size: 24px;
  font-weight: 700;
  color: v.$text-primary;
  margin-bottom: 4px;
}

.stat-card-trend {
  font-size: 13px;

  &.trend-up { color: var(--el-color-success); }
  &.trend-down { color: var(--el-color-danger); }
}

// ============================================================
// Panels
// ============================================================
.panel {
  background: v.$bg-card;
  border-radius: v.$border-radius-lg;
  box-shadow: v.$shadow-sm;
  overflow: hidden;
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid v.$border-color-light;
}

.panel-title {
  font-size: 15px;
  font-weight: 600;
  color: v.$text-primary;
}

.panel-body {
  padding: 20px;

  &--flush {
    padding: 0;
  }
}

// ============================================================
// Chart
// ============================================================
.chart-row {
  margin-bottom: 20px;
}

.chart-body {
  padding: 12px 12px 4px 4px;
}

.chart {
  width: 100%;
  height: 320px;
}

.chart-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 320px;
  padding: 40px;
}

// ============================================================
// Summary Items
// ============================================================
.summary-item {
  padding: 14px 0;
  border-bottom: 1px solid v.$border-color-light;

  &:last-child {
    border-bottom: none;
    padding-bottom: 0;
  }

  &:first-child {
    padding-top: 0;
  }
}

.summary-label {
  font-size: 13px;
  color: v.$text-secondary;
}

.summary-value {
  font-size: 20px;
  font-weight: 700;
  color: v.$text-primary;
  margin: 4px 0;
}

.summary-trend {
  font-size: 13px;

  .trend-up { color: var(--el-color-success); }
  .trend-down { color: var(--el-color-danger); }
}

.summary-compare {
  color: v.$text-placeholder;
  margin-left: 4px;
}

// ============================================================
// Bottom sections
// ============================================================
.bottom-row {
  margin-bottom: 20px;
}

.todo-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
}

.todo-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 20px;
  border-bottom: 1px solid v.$border-color-light;
  border-right: 1px solid v.$border-color-light;

  &:nth-child(3n) {
    border-right: none;
  }
}

.todo-label {
  font-size: 13px;
  color: v.$text-regular;
}

// ============================================================
// Overview Grid
// ============================================================
.overview-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px 12px;
}

.overview-item {
  text-align: center;
}

.overview-value {
  font-size: 26px;
  font-weight: 700;
  margin-bottom: 4px;
}

.overview-label {
  font-size: 13px;
  color: v.$text-secondary;
}
</style>
