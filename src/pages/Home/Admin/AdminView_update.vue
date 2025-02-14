<template>
  <div class="container mx-auto p-4">
    <!-- Button to update all data -->
    <div class="mb-4 text-right">
      <button 
        @click="updateAllData" 
        class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
      >
        Cập Nhật Dữ Liệu
      </button>
    </div>

    <h1 class="text-2xl font-bold mb-4">Revenue Dashboard</h1>

    <!-- Inputs for API 1: Monthly Revenue -->
    <div class="mb-4">
      <label class="block mb-2">Year for Monthly Revenue</label>
      <input 
        v-model="monthlyRevenueYear" 
        type="number" 
        class="border p-2 w-full"
        @change="fetchMonthlyRevenue"
      />
    </div>

    <!-- Inputs for API 2: Daily Revenue -->
    <div class="mb-4">
      <label class="block mb-2">Month and Year for Daily Revenue</label>
      <div class="flex space-x-2">
        <input 
          v-model="dailyRevenueMonth" 
          type="number" 
          min="1" 
          max="12" 
          placeholder="Month"
          class="border p-2 w-1/2"
        />
        <input 
          v-model="dailyRevenueYear" 
          type="number" 
          placeholder="Year"
          class="border p-2 w-1/2"
          @change="fetchDailyRevenue"
        />
      </div>
    </div>

    <!-- Inputs for API 3: Date Range Revenue -->
    <div class="mb-4">
      <label class="block mb-2">Date Range for Revenue</label>
      <div class="flex space-x-2">
        <input 
          v-model="startDate" 
          type="datetime-local" 
          class="border p-2 w-1/2"
        />
        <input 
          v-model="endDate" 
          type="datetime-local" 
          class="border p-2 w-1/2"
          @change="fetchDateRangeRevenue"
        />
      </div>
    </div>

    <!-- Charts Container -->
    <div class="grid grid-cols-2 gap-4">
      <!-- Monthly Revenue Chart -->
      <div>
        <h2 class="text-xl font-semibold mb-2">Monthly Revenue</h2>
        <canvas ref="monthlyRevenueChart"></canvas>
      </div>

      <!-- Daily Revenue Chart -->
      <div>
        <h2 class="text-xl font-semibold mb-2">Daily Revenue</h2>
        <canvas ref="dailyRevenueChart"></canvas>
      </div>

      <!-- Date Range Revenue Chart -->
      <div>
        <h2 class="text-xl font-semibold mb-2">Date Range Revenue</h2>
        <canvas ref="dateRangeRevenueChart"></canvas>
      </div>

      <!-- Order Statistics -->
      <div>
        <h2 class="text-xl font-semibold mb-2">Order Statistics</h2>
        <canvas ref="orderStatisticsChart"></canvas>
      </div>

      <!-- Brand Product Count -->
      <div>
        <h2 class="text-xl font-semibold mb-2">Product Count by Brand</h2>
        <canvas ref="brandProductChart"></canvas>
      </div>

      <!-- Category Product Count -->
      <div>
        <h2 class="text-xl font-semibold mb-2">Product Count by Category</h2>
        <canvas ref="categoryProductChart"></canvas>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import Chart from 'chart.js/auto'

// Refs for input
const monthlyRevenueYear = ref(2024)
const dailyRevenueMonth = ref(1)
const dailyRevenueYear = ref(2024)
const startDate = ref('')
const endDate = ref('')

// Chart references
const monthlyRevenueChart = ref(null)
const dailyRevenueChart = ref(null)
const dateRangeRevenueChart = ref(null)
const orderStatisticsChart = ref(null)
const brandProductChart = ref(null)
const categoryProductChart = ref(null)

// Fetch and process monthly revenue
const fetchMonthlyRevenue = async () => {
  try {
    const response = await axios.get(`/api/statistics/revenue/monthly?year=${monthlyRevenueYear.value}`)
    const monthlyData = Array(12).fill(0)
    
    response.data.forEach(item => {
      monthlyData[item.month - 1] = item.revenue
    })

    new Chart(monthlyRevenueChart.value, {
      type: 'bar',
      data: {
        labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
        datasets: [{
          label: 'Monthly Revenue',
          data: monthlyData,
          backgroundColor: 'rgba(75, 192, 192, 0.6)'
        }]
      }
    })
  } catch (error) {
    console.error('Error fetching monthly revenue:', error)
  }
}

// Fetch and process daily revenue
const fetchDailyRevenue = async () => {
  try {
    const response = await axios.get(`/api/statistics/revenue/daily?month=${dailyRevenueMonth.value.toString().padStart(2, '0')}&year=${dailyRevenueYear.value}`)
    const dailyData = Array(31).fill(0)
    
    response.data.forEach(item => {
      const date = new Date(item.date)
      const dayOfMonth = date.getDate() - 1
      dailyData[dayOfMonth] = item.revenue
    })

    new Chart(dailyRevenueChart.value, {
      type: 'bar',
      data: {
        labels: Array.from({length: 31}, (_, i) => i + 1),
        datasets: [{
          label: 'Daily Revenue',
          data: dailyData,
          backgroundColor: 'rgba(255, 99, 132, 0.6)'
        }]
      }
    })
  } catch (error) {
    console.error('Error fetching daily revenue:', error)
  }
}

// Fetch and process date range revenue
const fetchDateRangeRevenue = async () => {
  try {
    const response = await axios.get(`/api/orders/revenue`, {
      params: {
        startDate: startDate.value,
        endDate: endDate.value
      }
    })
    const rangeData = response.data.map(item => (item.revenue))

    new Chart(dateRangeRevenueChart.value, {
      type: 'bar',
      data: {
        labels: response.data.map(item => new Date(item.date).toLocaleDateString()),
        datasets: [{
          label: 'Revenue by Date',
          data: rangeData,
          backgroundColor: 'rgba(54, 162, 235, 0.6)'
        }]
      }
    })
  } catch (error) {
    console.error('Error fetching date range revenue:', error)
  }
}

// Fetch and process order statistics
const fetchOrderStatistics = async () => {
  try {
    const response = await axios.get('/api/statistics/orders')
    const { cancelledOrders, completedOrders, pendingOrders } = response.data

    new Chart(orderStatisticsChart.value, {
      type: 'pie',
      data: {
        labels: ['Cancelled', 'Completed', 'Pending'],
        datasets: [{
          data: [cancelledOrders, completedOrders, pendingOrders],
          backgroundColor: ['rgba(255, 99, 132, 0.6)', 'rgba(75, 192, 192, 0.6)', 'rgba(255, 205, 86, 0.6)']
        }]
      }
    })
  } catch (error) {
    console.error('Error fetching order statistics:', error)
  }
}

// Fetch and process brand product count
const fetchBrandProductCount = async () => {
  try {
    const response = await axios.get('/api/brands/product-count')
    new Chart(brandProductChart.value, {
      type: 'bar',
      data: {
        labels: response.data.map(item => item.brandName),
        datasets: [{
          label: 'Product Count by Brand',
          data: response.data.map(item => item.productCount),
          backgroundColor: 'rgba(153, 102, 255, 0.6)'
        }]
      }
    })
  } catch (error) {
    console.error('Error fetching brand product count:', error)
  }
}

// Fetch and process category product count
const fetchCategoryProductCount = async () => {
  try {
    const response = await axios.get('/api/products/category/product-count')
    new Chart(categoryProductChart.value, {
      type: 'bar',
      data: {
        labels: response.data.map(item => item.categoryName),
        datasets: [{
          label: 'Product Count by Category',
          data: response.data.map(item => item.productCount),
          backgroundColor: 'rgba(255, 159, 64, 0.6)'
        }]
      }
    })
  } catch (error) {
    console.error('Error fetching category product count:', error)
  }
}

// Update all charts
const updateAllData = () => {
  fetchMonthlyRevenue()
  fetchDailyRevenue()
  fetchDateRangeRevenue()
  fetchOrderStatistics()
  fetchBrandProductCount()
  fetchCategoryProductCount()
}

// Fetch all data when component mounts
onMounted(() => updateAllData())
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
}
</style>
