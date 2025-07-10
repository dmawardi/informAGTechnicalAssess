<!-- 
  PressureChart Component
  Displays a responsive chart widget with pressure data over time
  Features:
  - Chart type selector (line/bar)
  - Current pressure reading display
  - 24-hour trend indicator
  - Dynamic chart rendering based on selected type
-->
<template>
  <!-- Main container with padding, white background, rounded corners and shadow -->
  <div class="p-4 bg-white rounded-xl shadow w-10/12 mx-auto">
    <!-- Header section with title, current reading, and chart type selector -->
    <div class="flex justify-between items-center mb-4">
      <!-- Left side: Title and current pressure info -->
      <div>
        <h2 class="text-lg font-bold text-black">Pressure Over Time</h2>
        <!-- Current pressure reading - hardcoded for demo -->
        <p class="text-3xl font-bold text-black">{{ currentPressure }}</p>
        <!-- 24-hour trend indicator with positive change styling -->
        <p class="text-black">
          Last 24 Hours
          <span v-if="currentPressure >= pressure24HoursAgo" class="text-green-600">
            +{{ pressurePercentageChange }}%
          </span>
          <span v-else class="text-red-600"> -{{ pressurePercentageChange }}% </span>
        </p>
      </div>
      <!-- Right side: Chart type selector dropdown -->
      <select v-model="chartType" class="border rounded px-2 py-1 text-black">
        <option value="line">Line</option>
        <option value="bar">Bar</option>
      </select>
    </div>
    <!-- Chart container -->
    <div class="w-full py-4">
      <!-- Conditional chart rendering based on selected chart type -->
      <!-- Line chart component - shown when chartType is 'line' -->
      <Line v-if="chartType === 'line'" :data="chartData" :options="chartOptions" />
      <!-- Bar chart component - shown when chartType is not 'line' -->
      <Bar v-else :data="chartData" :options="chartOptions" />
    </div>
  </div>
</template>

<script setup>
// Vue 3 Composition API imports
import { ref, onMounted, computed } from 'vue'
// Vue Chart.js components for rendering line and bar charts
import { Line, Bar } from 'vue-chartjs'
// Chart.js core components and plugins
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  LineElement,
  BarElement,
  CategoryScale, // X-axis scale for categories
  LinearScale, // Y-axis scale for numeric values
  PointElement, // Data point markers for line charts
} from 'chart.js'
// Import inspection data service for fetching pump inspection records
import inspectionService from '@/services/inspectionService'

// Register Chart.js components globally
ChartJS.register(
  Title,
  Tooltip,
  Legend,
  LineElement,
  BarElement,
  CategoryScale,
  LinearScale,
  PointElement,
)

// Reactive state to allow dynamic updates
const inspectionData = ref([])
const chartType = ref('line')
const pressureChangeType = ref('positive') // Track if pressure change is positive or negative

// COMPUTED PROPERTIES
//
// Chart labels and data points
const labels = computed(() => {
  return inspectionData.value
    .slice(0, 24) // Get last 24 hours
    .reverse() // Reverse to show oldest to newest on chart
    .map((record) => {
      // Format datetime to show only hour:minute (e.g., "09:00")
      const date = new Date(record.datetime)
      return date.toLocaleTimeString('en-US', {
        hour: '2-digit',
        minute: '2-digit',
        hour12: false,
      })
    })
})
const dataPoints = computed(() => {
  return inspectionData.value
    .slice(0, 24) // Get last 24 hours
    .reverse() // Reverse to show oldest to newest on chart
    .map((record) => record.pressure)
})
// Computed Pressure values
const currentPressure = computed(() => {
  return inspectionData.value.length > 0 ? inspectionData.value[0].pressure : 0
})
const pressure24HoursAgo = computed(() => {
  return inspectionData.value.length > 23 ? inspectionData.value[23].pressure : 0
})
// Calculate percentage change from current pressure to 24 hours ago
const pressurePercentageChange = computed(() => {
  if (pressure24HoursAgo.value === 0) return 0 // Avoid division by zero
  let changeInPressure =
    ((currentPressure.value - pressure24HoursAgo.value) / pressure24HoursAgo.value) * 100
  if (changeInPressure < 0) {
    changeInPressure = Math.abs(changeInPressure) // Ensure positive value for display
    pressureChangeType.value = 'negative' // Set type for negative change
  } else {
    pressureChangeType.value = 'positive' // Set type for positive change
  }
  return changeInPressure.toFixed(2) // Format to 2 decimal places
})

// Chart data configuration
//
const chartData = computed(() => ({
  labels: labels.value, // X-axis labels (time)
  datasets: [
    {
      label: 'Pressure (PSI)', // Dataset label for tooltips
      data: dataPoints.value, // Y-axis data points
      fill: false, // Don't fill area under line
      borderColor: '#3b82f6', // Line/bar border color (blue)
      backgroundColor: '#3b82f6', // Fill/bar background color
      tension: 1, // Line curve smoothness (0 = straight, 1 = very curved)
    },
  ],
}))
// Chart configuration options (controls appearance, behavior, and scaling of the chart)
const chartOptions = ref({
  responsive: true, // Chart resizes with container
  maintainAspectRatio: false, // Allow chart to resize freely
  plugins: {
    legend: { display: false }, // Hide legend since we have custom title
  },
  scales: {
    y: {
      // Y-axis configuration
      beginAtZero: true, // Start Y-axis at 0
      title: {
        display: true,
        text: 'Pressure (PSI)', // Y-axis label
      },
    },
    x: {
      // X-axis configuration
      title: {
        display: true,
        text: 'Time', // X-axis label
      },
    },
  },
})

// Get inspection data from service on component mount
onMounted(() => {
  // Fetch data for pump ID 1
  inspectionData.value = inspectionService.getInspectionsByPump(1)
})
</script>

<style scoped></style>
