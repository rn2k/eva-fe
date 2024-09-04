<template>
  <div class="chart-container">
    <Bar :data="chartData" :options="chartOptions" />
  </div>
</template>

<script>
import { Bar } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

export default {
  components: {
    Bar
  },
  props: {
    latitudes: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      chartData: {
        labels: this.generateLabels(this.latitudes.length * 2),
        datasets: [
          {
            label: 'Daily Sales',
            backgroundColor: '#42A5F5',
            data: this.generateData(this.latitudes, this.latitudes.length * 2) 
          }
        ]
      },
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          x: {
            beginAtZero: true,
            title: {
              display: true,
              text: 'Day'
            }
          },
          y: {
            beginAtZero: true,
            title: {
              display: true,
              text: 'Amount ($)'  
            },
            ticks: {
              callback: function(value) {
                return value + '%'; 
              }
            }
          }
        }
      }
    };
  },
  methods: {
    generateLabels(count) {
      return Array.from({ length: count }, (_, index) => `Day ${index + 1}`);
    },
    generateData(dataArray, totalCount) {
      const repeatedData = [...dataArray, ...dataArray];
      return repeatedData.slice(0, totalCount); 
    }
  },
  watch: {
    latitudes(newLatitudes) {
      if (newLatitudes) {
        this.chartData.labels = this.generateLabels(newLatitudes.length * 2);
        this.chartData.datasets[0].data = this.generateData(newLatitudes, newLatitudes.length * 2);
      }
    }
  }
};
</script>

<style scoped>
.chart-container {
  position: relative;
  width: 100%;
  height: 500px;
}
</style>
