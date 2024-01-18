
<template>
  <div>
    <!-- Display a message if the graph is not ready or data is not available -->
    <p v-if="showGraph && chartData.labels.length && chartData.datasets.length">
      Chart will be displayed here
    </p>
    <div v-else>
      <!-- Render the Bar chart if data is available -->
      <Bar :data="barChartData" :options="chartOptions" />
    </div>
  </div>
</template>

<script>
import { Bar } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'
ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

export default {
  components: { Bar },
  props: {
    // Data for rendering the chart
    chartData: {
      type: Object,
      default: () => ({ labels: [], datasets: [] }),
    },
    // Title of the questionnaire for determining color scales
    questionnaireTitle: {
      type: String,
      default: "", // Provide a default value for questionnaireTitle
    },
  },
  data() {
    return {
      // Flag to control the visibility of the graph
      showGraph: false,
    };
  },

  computed: {
    /**
     * @vuese
     * Prepare data for rendering the Bar chart.
     * @returns {Object} - Formatted data for Bar chart.
     */
    barChartData() {
      return {
        labels: this.chartData.labels,
        datasets: [
          {
            data: this.chartData.datasets[0].data,
            backgroundColor: this.barColors,
          },
        ],
      };
    },
    /**
     * @vuese
     * Determine colors for each bar in the chart based on total scores.
     * @returns {Array} - Array of colors for each bar.
     */
    barColors() {
      const colorScales = {
        'Oxford Knee Score': { 19: '#d32f2f', 29: '#ffa500', 39: '#ffeb3b', 48: '#8bae3b' },
        'Oxford Hip Score': { 19: '#d32f2f', 29: '#ffa500', 39: '#ffeb3b', 48: '#8bae3b' },
        'Neck Disability Index': { 4: '#8bae3b', 14: 'lightgreen', 24: '#ffeb3b', 34: '#ffa500', 50: '#d32f2f' },
        'Oswestry Disability Index': { 4: '#8bae3b', 14: 'lightgreen', 24: '#ffeb3b', 34: '#ffa500', 50: '#d32f2f' },
      };

      return this.chartData.labels.map((label, index) => {
        const totalScore = this.chartData.datasets[0].data[index];

        let color = 'white';
        if (colorScales[this.questionnaireTitle]) {
          for (const [threshold, scaleColor] of Object.entries(colorScales[this.questionnaireTitle])) {
            if (totalScore <= parseInt(threshold)) {
              color = scaleColor;
              break;
            }
          }
        }

        return color;
      });
    },
    /**
     * @vuese
     * Chart options configuration.
     * @returns {Object} - Options for the Bar chart.
     */
     chartOptions() {
      // Set default max Y-axis value for 'Oxford Knee Score' and 'Oxford Hip Score'
      let maxYAxis = 48;

      // Adjust max Y-axis value for 'Neck Disability Index' and 'Oswestry Disability Index'
      if (['Neck Disability Index', 'Oswestry Disability Index'].includes(this.questionnaireTitle)) {
        maxYAxis = 50;
      }

      return {
        scales: {
          x: {
            grid: {
              display: false,
            },
          },
          y: {
            grid: {
              display: false,
            },
            max: maxYAxis, // Set the max value for the Y-axis
            title: {
              display: true,
              text: 'Total Score', // Add a title to the Y-axis
            },
          },
        },
        plugins: {
          legend: {
            display: false,
          },
        },
      };
    },
  }, 

  mounted() {
    console.log("TotalScoreGraph Receive Data", this.chartData);
  },
};
</script>
 