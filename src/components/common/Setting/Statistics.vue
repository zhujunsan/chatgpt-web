<script setup lang='ts'>
import { onMounted, ref } from 'vue'
import {NCol, NDatePicker, NIcon, NNumberAnimation, NRow, NSpin, NStatistic} from 'naive-ui'
import { BarChart } from 'vue-chart-3'
import { Chart, registerables } from 'chart.js'
import { t } from '@/locales'
import { fetchUserStatistics } from '@/api'
import {SvgIcon} from "@/components/common";

Chart.register(...registerables)

const chartData = ref({
  labels: [],
  datasets: [
    {
      label: t('setting.statisticsPrompt'),
      data: [],
      borderColor: '#a1dc95',
      backgroundColor: '#a1dc95',
      stack: 'Usage',
    },
    {
      label: t('setting.statisticsCompletion'),
      data: [],
      borderColor: '#6d6e7e',
      backgroundColor: '#6d6e7e',
      stack: 'Usage',
    },
  ],
})

const chartConfig = ref({
  type: 'bar',
  data: chartData,
  options: {
    responsive: true,
    scales: {
      x: {
        stacked: true,
      },
      y: {
        stacked: true,
      },
    },
  },
})

const summary = ref({
  promptTokens: 0,
  completionTokens: 0,
  totalTokens: 0,
})
const statisticsChart = ref()
const loading = ref(false)
const range = ref<[number, number]>([
  new Date().getTime() - 30 * 86400 * 1000,
  new Date().getTime(),
])

async function fetchStatistics() {
  try {
    loading.value = true
    const { data } = await fetchUserStatistics(...range.value)

    if (Object.keys(data.chartData).length) {
      summary.value.promptTokens = data.promptTokens
      summary.value.completionTokens = data.completionTokens
      summary.value.totalTokens = data.totalTokens

      chartData.value.labels = data.chartData.map((item: any) => item._id)
      chartData.value.datasets[0].data = data.chartData.map((item: any) => item.promptTokens)
      chartData.value.datasets[1].data = data.chartData.map((item: any) => item.completionTokens)
      statisticsChart.value.update()
    }
  }
  finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchStatistics()
})
</script>

<template>
  <NSpin :show="loading">
    <div class="p-4 space-y-5 min-h-[200px]">
      <div class="space-y-6">
        <div class="flex items-center space-x-4">
          <span class="flex-shrink-0 w-[100px]">{{ $t('setting.statisticsPeriod') }}</span>
          <div class="flex-1">
            <NDatePicker
              v-model:value="range"
              type="datetimerange"
              clearable
              @update:value="fetchStatistics"
            />
          </div>
        </div>

        <div class="flex items-center space-x-4">
          <NRow>
            <NCol :span="8" class="text-center">
              <NStatistic :label="t('setting.statisticsPrompt')">
                <template #prefix>
                  <NIcon>
                    <SvgIcon class="text-lg" icon="ri-chat-upload-line" />
                  </NIcon>
                </template>
                <NNumberAnimation :from="0" :to="summary.promptTokens" />
              </NStatistic>
            </NCol>
            <NCol :span="8" class="text-center">
              <NStatistic :label="t('setting.statisticsCompletion')">
                <template #prefix>
                  <NIcon>
                    <SvgIcon class="text-lg" icon="ri-chat-download-line" />
                  </NIcon>
                </template>
                <NNumberAnimation :from="0" :to="summary.completionTokens" />
              </NStatistic>
            </NCol>
            <NCol :span="8" class="text-center">
              <NStatistic :label="t('setting.statisticsTotal')">
                <template #prefix>
                  <NIcon>
                    <SvgIcon class="text-lg" icon="ri-question-answer-line" />
                  </NIcon>
                </template>
                <NNumberAnimation :from="0" :to="summary.totalTokens" />
              </NStatistic>
            </NCol>
          </NRow>
        </div>

        <BarChart
          style="aspect-ratio: 3/2;"
          v-show="chartData.labels.length"
          ref="statisticsChart"
          :config="chartConfig"
          :chart-data="chartData"
        />
      </div>
    </div>
  </NSpin>
</template>

<style>

</style>
