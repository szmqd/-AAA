<template>
  <div class="app-container">
    <ve-histogram
      :extend="chartExtend"
      :data="chartData"
      :settings="chartSettings"
    />
  </div>
</template>

<script>
import { statGoods } from "@/api/stat";
import VeHistogram from "v-charts/lib/line";

export default {
  components: { VeHistogram },
  data() {
    return {
      chartData: {},
      chartSettings: {},
      chartExtend: {}
    };
  },
  created() {
    statGoods().then(response => {
      this.chartData = response.data.data;
      this.chartSettings = {
        labelMap: {
          orders: "订单量",
          products: "下单货品数量",
          amount: "下单货品总额"
        }
      };
      this.chartExtend = {
        xAxis: { boundaryGap: true },
        series: {
          label: { position: "top" }
        }
      };
    });
  }
};
</script>
