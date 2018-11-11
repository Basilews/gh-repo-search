<script>
  import { Bar } from 'vue-chartjs';

  export default {
    extends: Bar,
    name: 'bar-chart',
    props: {
      contributors: Array
    },
    mounted() {
      this.renderBarChart();
    },
    computed: {
      chartData: function() {
        return this.contributors;
      }
    },
    methods: {
      renderBarChart: function() {
        this.renderChart({
          labels: this.chartData.map(person => person.login),
          datasets: [
            {
              label: 'Contributions',
              backgroundColor: '#f87979',
              data: this.chartData.map(person => person.contributions)
            }
          ],
        },
        {
          height: '400px',
          responsive: true,
          legend: {
            labels: {
              fontColor: 'white',
            }
          },
          scales: {
            yAxes: [{
              ticks: {
                fontColor: "white",
                min: 0,
                beginAtZero: true,
                callback: function(value, index, values) {
                    if (Math.floor(value) === value) {
                        return value;
                    }
                }
              }
            }],
            xAxes: [{
              ticks: {
                fontColor: "white",
                autoSkip: false,
              }
            }]
          },
        })
      }
    },
    watch: {
      contributors: function() {
        this.renderBarChart();
      }
    }
  }
</script>
