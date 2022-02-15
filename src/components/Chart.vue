<script>
import { Line } from 'vue3-chart-v2'

export default {
  extends: Line,
  props: {
    dataProp: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      data: this.dataProp,
      gradient: null,
    }
  },
  watch: {
    dataProp: {
      deep: true,
      handler(newVal) {
        this.data = newVal
        this.createChart()
      },
    },
  },
  methods: {
    cancelAnimation() {
      for (const chart of Object.values(window.Chart.instances)) {
        window.Chart.animationService.cancelAnimation(chart)
      }
    },
    gradients(hex) {
      this.gradient = this.$refs.canvas
        .getContext('2d')
        .createLinearGradient(0, 0, 0, 450)

      this.gradient.addColorStop(0, `${hex}80`)
      this.gradient.addColorStop(1, `${hex}00`)
    },
    async createChart() {
      const chart = this?.state?.chartObj?.chart

      if (chart) {
        this.cancelAnimation()
        this.gradient = null
        await chart.destroy()
      }

      const { team, labels, set_1, set_2, axes } = this.data

      this.gradients(team.color_secondary)

      let datasets = [
        {
          label: set_1.label,
          borderColor: team.color_secondary,
          pointBackgroundColor: team.color_secondary,
          borderWidth: 1,
          pointBorderColor: team.color_secondary,
          backgroundColor: this.gradient,
          data: set_1.data,
        },
      ]

      if (set_2?.data) {
        datasets.push({
          label: set_2.label,
          borderColor: 'white',
          pointBackgroundColor: 'white',
          borderWidth: 1,
          pointBorderColor: 'white',
          backgroundColor: this.gradient,
          data: set_2.data,
        })
      }

      let chartCore = {
        labels: labels,
        datasets: datasets,
      }

      let chartOptions = {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          yAxes: [{ scaleLabel: { display: true, labelString: axes.y } }],
          xAxes: [{ scaleLabel: { display: true, labelString: axes.x } }],
        },
      }

      this.renderChart(chartCore, chartOptions)
    },
  },
}
</script>
