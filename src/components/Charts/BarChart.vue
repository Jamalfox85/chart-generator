<template>
  <div>
    <div class="chart-controls" v-if="spreadsheetHeaders.length > 0">
      <h2>Chart Controls</h2>
      <div class="flex my-4">
        <div class="control-group mr-2">
          <label for="x-axis">X-Axis</label>
          <n-select v-model:value="xAxisMetric" :options="xAxisOptions" />
        </div>
        <div class="control-group mr-2">
          <label for="y-axis">Y-Axis</label>
          <n-select v-model:value="yAxisMetric" :options="yAxisOptions" />
        </div>
      </div>
    </div>
    <div ref="chartContainer"></div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch, toRefs } from 'vue'
import { NSelect } from 'naive-ui'
import * as d3 from 'd3'

const props = defineProps<{
  spreadsheetData: any[]
}>()

const { spreadsheetData } = toRefs(props)
let spreadsheetHeaders = spreadsheetData.value[0] || []
let xAxisOptions = ref<{ label: string; value: number }[]>([])
let yAxisOptions = ref<{ label: string; value: number }[]>([])
let xAxisMetric = ref<number>(0)
let yAxisMetric = ref<number>(0)

const chartContainer = ref<HTMLElement | null>(null)
function createChart(data: any[]) {
  if (data.length === 0) return

  const width = chartContainer.value?.clientWidth || 640
  const height = 400
  const marginTop = 20
  const marginRight = 20
  const marginBottom = 20
  const marginLeft = 50

  // Declare the x (horizontal position) scale.
  const x = d3
    .scaleBand()
    .domain(
      d3.groupSort(
        data,
        ([d]: any) => -d[yAxisMetric.value],
        (d: any) => d[xAxisMetric.value]
      )
    ) // descending frequency
    .range([marginLeft, width - marginRight])
    .padding(0.1)

  // Declare the y (vertical position) scale.
  const y = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d: any) => d[yAxisMetric.value])])
    .range([height - marginBottom, marginTop])

  // Create the SVG container.
  const svg = d3
    .create('svg')
    .attr('width', width)
    .attr('height', height)
    .attr('viewBox', [0, 0, width, height])
    .attr('style', 'max-width: 100%; height: auto;')

  // Add a rect for each bar.
  svg
    .append('g')
    .attr('fill', 'steelblue')
    .selectAll('rect')
    .data(data)
    .join('rect')
    .attr('x', (d: any) => x(d[xAxisMetric.value]))
    .attr('y', (d: any) => y(d[yAxisMetric.value]))
    .attr('height', (d: any) => y(0) - y(d[yAxisMetric.value]))
    .attr('width', x.bandwidth())

  // Add the x-axis and label.
  svg
    .append('g')
    .attr('transform', `translate(0,${height - marginBottom})`)
    .call(d3.axisBottom(x).tickSizeOuter(0))

  // Add the y-axis and label, and remove the domain line.
  svg
    .append('g')
    .attr('transform', `translate(${marginLeft},0)`)
    .call(
      d3.axisLeft(y).tickFormat((y: any) => {
        if (y >= 1000) {
          return `${y / 1000}k`
        } else if (y < 1) {
          return y * 100 + '%' // Convert to percentage
        }
        return y.toFixed()
      })
    )
    .call((g: any) => g.select('.domain').remove())
    .call((g: any) =>
      g
        .append('text')
        .attr('x', -marginLeft)
        .attr('y', 10)
        .attr('fill', 'currentColor')
        .attr('text-anchor', 'start')
        .text(`${spreadsheetHeaders[yAxisMetric.value]}`)
    )
  // Append the chart to the container
  const chart = svg.node()
  if (chart && chartContainer.value) {
    chartContainer.value.innerHTML = '' // Clear previous chart
    chartContainer.value.appendChild(chart)
  }
}

onMounted(() => {
  // Run the chart creation initially on mount if data exists
  if (spreadsheetData.value.length > 0) {
    createChart(spreadsheetData.value.slice(1)) // Exclude headers
  }
})

// Watch for changes in the spreadsheet data and update the chart accordingly
watch(spreadsheetData, (newData) => {
  spreadsheetHeaders = spreadsheetData.value[0] || []
  xAxisOptions = spreadsheetHeaders
    .map((header, originalIndex) => {
      if (typeof newData[1][originalIndex] === 'string') {
        return { label: header, value: originalIndex }
      }
      return null
    })
    .filter(Boolean)

  yAxisOptions = spreadsheetHeaders
    .map((header, originalIndex) => {
      if (typeof newData[1][originalIndex] === 'number') {
        return { label: header, value: originalIndex }
      }
      return null
    })
    .filter(Boolean)

  xAxisMetric.value = xAxisOptions[0]?.value || 0
  yAxisMetric.value = yAxisOptions[0]?.value || 0
  console.log(newData)
  createChart(newData.slice(1)) // Skip headers
})
watch([xAxisMetric, yAxisMetric], ([newX, newY]) => {
  createChart(spreadsheetData.value.slice(1))
})
</script>

<style lang="scss">
.n-base-select-menu {
  width: fit-content;
}
</style>
