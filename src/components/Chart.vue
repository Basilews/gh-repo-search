<script>
  import { Line } from 'vue-chartjs';

  export default {
    extends: Line,
    name: 'line-chart',
    props: {
      contributors: Array
    },
    mounted () {
      // Overwriting base render method with actual data.
      this.renderChart({
        labels: this.contributors.map(person => person.login),
        datasets: [
          {
            label: 'Contributions',
            backgroundColor: '#f87979',
            data: this.contributors.map(person => person.contributions)
          }
        ],
      },
      {
        legend: {
          labels: {
            fontColor: 'white',
          }
        },
        scales: {
          yAxes: [{
            ticks: {
              fontColor: "white",
            }
          }],
          xAxes: [{
            ticks: {
              fontColor: "white",
            }
          }]
        },
      })
    },
  }
</script>

<style lang="sass" scoped>

</style>
