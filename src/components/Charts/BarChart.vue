<template>
  <div>
    <div ref="chartContainer"></div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch, toRefs } from 'vue'
import * as d3 from 'd3'

const props = defineProps<{
  spreadsheetData: any[]
}>()
const { spreadsheetData } = toRefs(props)

const spreadsheetHeaders = spreadsheetData.value[0]
watch(spreadsheetData, (newData) => {
  let data = newData.slice(1) // Skip first row (headers)
  // console.log('New data:', data)
  const width = 640
  const height = 400
  const marginTop = 20
  const marginRight = 20
  const marginBottom = 20
  const marginLeft = 20

  // Declare the x (horizontal position) scale.
  const x = d3
    .scaleBand()
    .domain(
      d3.groupSort(
        data,
        ([d]: any) => -d[9],
        (d: any) => d[2]
      )
    ) // descending frequency
    .range([marginLeft, width - marginRight])
    .padding(0.1)

  // Declare the y (vertical position) scale.
  const y = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d: any) => d[9])])
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
    .selectAll()
    .data(data)
    .join('rect')
    .attr('x', (d: any) => x(d[2]))
    .attr('y', (d: any) => y(d[9]))
    .attr('height', (d: any) => y(0) - y(d[9]))
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
    .call(d3.axisLeft(y).tickFormat((y: any) => (y * 100).toFixed()))
    .call((g: any) => g.select('.domain').remove())
    .call((g: any) =>
      g
        .append('text')
        .attr('x', -marginLeft)
        .attr('y', 10)
        .attr('fill', 'currentColor')
        .attr('text-anchor', 'start')
        .text('↑ Frequency (%)')
    )

  // Get the chart node (the SVG element)
  const chart = svg.node()

  // Create a ref to hold the chart container
  const chartContainer = ref(null)
  if (chart && chartContainer.value) {
    chartContainer.value.appendChild(chart)
  }
})

onMounted(() => {
  // Append the SVG chart to the container on mount
})
</script>

<style scoped></style>
