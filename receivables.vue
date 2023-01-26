<template>
  <div class="flex flex-col space-y-16">
    <div>
      <canvas ref="chart" style="margin: auto"></canvas>
    </div>
    <div class="pt-16">
      <table class="w-full">
        <tbody>
          <tr>
            <td>Private</td>
            <td>{{ privateAccountPer }} %</td>
            <td>₦ {{ totalPrivateAccount }}</td>
          </tr>
          <tr>
            <td>Self Funded</td>
            <td>{{ selfFunPer }} %</td>
            <td>₦ {{ totalSelfFunded }}</td>
          </tr>
          <tr>
            <td>Insurance</td>
            <td>{{ InsurancePer }} %</td>
            <td>₦ {{ totalInsurance }}</td>
          </tr>
          <tr class="font-bold">
            <td>Total</td>
            <td>100%</td>
            <td>₦ {{ totalCollectables }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<script lang="ts">
import { CustomDoughnutController } from "@/plugins/chart";
import { cornieClient } from "@/plugins/http";
import { Chart } from "chart.js";
import { Options, Vue } from "vue-class-component";
import { Prop, Watch } from "vue-property-decorator";
import ChartCard from "./chart-card.vue";

Chart.register(CustomDoughnutController);

@Options({
  name: "BillingsChart",
  components: {
    ChartCard,
  },
})
export default class BillingsChart extends Vue {
  chart!: Chart;

  get totalBillings() {
    return "₦ 17,782,000.00";
  }

  mounted() {
    this.mountChart();
  }

  get display() {
    return {
      key: "",
      percentage: "",
      total: "",
    };
  }

  @Prop({ type: Object, default: "" })
  range!: object;

  @Watch("range")
  rangeUpdated() {
    console.log("range", this.range);
    this.fetchData();
  }
  totalPrivateAccount = 0;
  totalSelfFunded = 0;
  totalInsurance = 0;
  totalCollectables = 0;

  get privateAccountPer() {
    let p = this.percentage(this.totalPrivateAccount, this.totalCollectables);
    return isNaN(p) ? 0 : p;
  }

  get selfFunPer() {
    let t = this.percentage(this.totalSelfFunded, this.totalCollectables);
    return isNaN(t) ? 0 : t;
  }

  get InsurancePer() {
    let r = this.percentage(this.totalInsurance, this.totalCollectables);
    return isNaN(r) ? 0 : r;
  }

  percentage(partialValue: number, totalValue: number) {
    return (100 * Number(partialValue)) / Number(totalValue);
  }

  sumTotalCollectables(obj: object) {
    return Object.values(obj).reduce((a, b) => a + b);
  }

  isEmpty = true;
  async fetchData() {
    try {
      const { start, end } = this.range as any;
      if (!start || !end) return;
      const startDate = new Date(start).toISOString();
      const endDate = new Date(end).toISOString();
      const response = await cornieClient().get(
        `api/v1/bill/stat/receivables?start=${startDate}&end=${endDate}`
      );
      const { privateAccount, selfFunded, insurance } = response.data;
      this.totalCollectables = this.sumTotalCollectables(response.data);
      this.totalPrivateAccount = privateAccount;
      this.totalSelfFunded = selfFunded;
      this.totalInsurance = insurance;
      this.isEmpty = false;
    } catch (error) {
      console.log("error", error);
      this.isEmpty = true;
      window.notify({ msg: "Failed to fetch chart data", status: "error" });
    }
  }
  get emptyChartData() {
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
  }

  get chartData() {
    return {
      labels: ["Private", "Self funded", "Insurance claims"],
      datasets: [
        {
          label: "My First Dataset",
          data: [
            this.totalPrivateAccount,
            this.totalSelfFunded,
            this.totalInsurance,
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
  }

  get chartText() {
    return `Total - ₦ ${this.totalCollectables}`;
  }

  mountChart() {
    const data = this.isEmpty == true ? this.emptyChartData : this.chartData;
    const ctx: any = this.$refs.chart;
    this.chart?.destroy();
    //@ts-ignore
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
          //@ts-ignore
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
</script>

<style scoped>
.rating-grid {
  grid-template-columns: 30% 70%;
}
.w200 {
  width: 200px !;
  height: 200px;
}
td {
  font-size: 14px !important;
  padding: 14px 16px !important;
  line-height: 20px;
  color: #14171f;
}
table > tbody > tr > td:last-child {
  display: flex;
  flex-direction: row;
  justify-content: flex-end !important;
}
table > tbody > tr:nth-of-type(even) {
  background: #f0f4fe !important;
  border-radius: 8px;
}
table,
tbody,
tr {
  border: none !important;
}
</style>
