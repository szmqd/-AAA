<template>
  <div class="app-container">
    <ve-line
      :extend="chartExtend"
      :data="chartData"
      :settings="chartSettings"
    />
  </div>
</template>

<script>
import { statOrder } from "@/api/stat";
import VeLine from "v-charts/lib/line";

export default {
  components: { VeLine },
  data() {
    return {
      chartData: {},
      chartSettings: {},
      chartExtend: {},
    };
  },
  created() {
    statOrder().then((response) => {
      this.chartData = response.data.data;
      this.chartSettings = {
        tooltip: {
          trigger: "item",
        },
        legend: {
          data: [
            { '订单量': orders },
            { '下单用户': customers },
            { '订单总额': amount },
            { '客单价': pcr },
          ],
        },
      };
      this.chartExtend = {
        xAxis: {
          type: "category",
          boundaryGap: false,
          data: [day],
        },
        series: {
          label: { show: true, position: "top" },
        },
      };
    });
  },
};
</script>