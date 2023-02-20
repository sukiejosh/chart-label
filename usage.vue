<template>
<div>
<canvas ref="chart" style="margin: auto"></canvas>
</div>

</template>
<script>
import { CustomDoughnutController } from './custom-chart.js'
import { Chart } from "chart.js";

Chart.register(CustomDoughnutController);
export default {
data(){
return {
isEmpty:false
}
}


  mounted() {
    this.mountChart();
  }
  
  computed:{
  emptyChartData() {
    return {
      labels: ["Private", "Self funded", "Insurance claims"],
      datasets: [
        {
          label: "Dataset 1",
          backgroundColor: ["#C2C7D6", "#114FF5", "#FE4D3C", "#35BA83"],
          data: [1, 0, 0, 0],
        },
      ],
    };
  },
  
  chartData() {
    return {
      labels: ["Private", "Self funded", "Insurance claims"],
      datasets: [
        {
          label: "My First Dataset",
          data: [
            1,1,0,1
          ],
          backgroundColor: [
            "rgb(255, 99, 132)",
            "rgb(54, 162, 235)",
            "rgb(255, 205, 86)",
          ],
          hoverOffset: 4,
        },
      ],
    };
  },
  chartText() {
    return `Total - ₦ ${100000}`;
  }

  }
  
  
  methods:{
  mountChart() {
    const data = this.isEmpty == true ? this.emptyChartData : this.chartData;
    const ctx: any = this.$refs.chart;
    this.chart.destroy();
    this.chart = new Chart(ctx, {
      type: "derivedDoughnut",
      data,
      options: {
        // cutout: 90,
        elements: {
          center: {
            text: this.chartText,
            color: "#14171F",
            fontStyle: "Inter",
            sidePadding: 5,
            minFontSize: 5,
            lineHeight: 5,
          },
        },
        cutout: "80%",
        responsive: true,
        plugins: {
          legend: false,
          // legend: {
          //   position: "bottom",
          //   labels: {
          //     boxWidth: 10,
          //     boxHeight: 10,
          //     pointStyle: "circle",
          //     usePointStyle: true,
          //   },
          // },
          title: {
            display: false,
          },
        },
      },
    });
    if (this.chart && this.chart.canvas.parentNode) {
      const parent: any = this.chart.canvas.parentNode;
      parent.style.height = "200px";
    }
  }
  
  }



}

</script>
