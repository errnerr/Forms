<template>
  <div class="border border-nt-blue-light bg-blue-50 dark:bg-notion-dark-light rounded-md p-4 mb-5 w-full mx-auto mt-4 select-all">
    <div v-if="!form.is_pro" class="relative">
      <div class="absolute inset-0 z-10">
        <div class="p-5 max-w-md mx-auto mt-5">
          <p class="text-center">
            You need a <pro-tag class="mx-1" /> subscription to access your form analytics.
          </p>
          <p class="mt-5 text-center">
            <v-button :to="{name:'pricing'}">
              Subscribe
            </v-button>
          </p>
        </div>
      </div>
      <img :src="asset('img/pages/forms/blurred_graph.png')"
           alt="Sample Graph"
           class="mx-auto filter blur-md z-0"
      >
    </div>
    <loader v-else-if="isLoading" class="h-6 w-6 text-nt-blue mx-auto" />
    <LineChart v-else
               :chart-options="chartOptions"
               :chart-data="chartData"
    />
  </div>
</template>

<script>
import axios from 'axios'
import { Line as LineChart } from 'vue-chartjs/legacy'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  LineElement,
  LinearScale,
  CategoryScale,
  PointElement
} from 'chart.js'
import ProTag from '../../../common/ProTag.vue'

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  LineElement,
  LinearScale,
  CategoryScale,
  PointElement
)

export default {
  name: 'FormStats',
  components: {
    ProTag,
    LineChart
  },
  props: {
    form: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      isLoading: true,
      chartData: {
        labels: [],
        datasets: [
          {
            label: 'Form Views',
            backgroundColor: 'rgba(59, 130, 246, 1)',
            borderColor: 'rgba(59, 130, 246, 1)',
            data: []
          },
          {
            label: 'Form Submissions',
            backgroundColor: 'rgba(16, 185, 129, 1)',
            borderColor: 'rgba(16, 185, 129, 1)',
            data: []
          }
        ]
      },
      chartOptions: {
        scales: {
          y: {
            beginAtZero: true,
            ticks: {
              precision: 0
            }
          }
        },
        responsive: true,
        maintainAspectRatio: false
      }
    }
  },
  mounted () {
    this.getChartData()
  },
  methods: {
    getChartData () {
      if (!this.form || !this.form.is_pro) { return null }
      this.isLoading = true
      axios.get('/api/open/workspaces/' + this.form.workspace_id + '/form-stats/' + this.form.id).then((response) => {
        const statsData = response.data
        if (statsData && statsData.views !== undefined) {
          this.chartData.labels = Object.keys(statsData.views)
          this.chartData.datasets[0].data = statsData.views
          this.chartData.datasets[1].data = statsData.submissions
          this.isLoading = false
        }
      })
    }
  }
}
</script>
