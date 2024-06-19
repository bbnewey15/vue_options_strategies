<template>
  <div>
    <h1>Options Profit Calculator</h1>

    <div id="chart">
      <apex-chart
        type="area"
        height="450"
        :options="chartOptions"
        :series="series"
      ></apex-chart>
    </div>
  </div>
</template>

<script>
import VueApexCharts from "vue-apexcharts";
import Vue from "vue";
Vue.use(VueApexCharts);

Vue.component("apex-chart", VueApexCharts);

export default {
  name: "CodingChallenge",
  props: ["optionsData"],
  data: function () {
    const generateData = (option) => {
      var name = option.type + " " + option.strike_price;
      var data = [];

      // we want x to be from strike_price - 10 to strike_price + 10
      // for a put, if long, the lowest y can go is the strike price, if short, y is  ((strike price) - (ask-bid)) * 100
      // for a call, if long, the lowest y can go is the strike price, if short, y is  ((strike price) + (ask-bid)) * 100

      //populate x and y values
      for (var i = 0; i <= option.strike_price + 50; i++) {
        var x = i;
        var y = 0;
        var current_price = (option.ask + option.bid) / 2;
        var difference = x - current_price;
        var profit_loss = 0;
        var highest,
          lowest = 0;
        if (option.type == "Put") {
          if (option.long_short == "long") {
            // Buying a put means the lowest y can go is the strike price
            // the highest y can go is the strike price + (ask - bid) * 100

            // if difference is negative then the option is in the money

            if (difference < 0) {
              profit_loss = Math.abs(difference) * 100;
              y = profit_loss;
            } else {
              profit_loss = difference * 100;
              // set negative
              profit_loss = 0 - profit_loss;
              y = Math.max(profit_loss, lowest);
            }
          } else {
            // Selling a put means the highest y can go is the strike price
            // the lowest y can go is the strike price + (ask - bid) * 100
            // if difference is negative then the option is in the money

            if (difference < 0) {
              profit_loss = difference * 100;
              // already negative
              y = profit_loss;
            } else {
              profit_loss = difference * 100;
              y = Math.min(profit_loss, highest);
            }
          }
        } else {
          if (option.long_short == "long") {
            // Buying a call means the lowest y can go is the strike price
            // the highest y can go is the strike price + (ask - bid) * 100
            // if difference is positive then the option is in the money

            if (difference < 0) {
              profit_loss = difference * 100;
              // already negative
              y = Math.max(profit_loss, lowest);
            } else {
              profit_loss = difference * 100;
              y = profit_loss;
            }
          } else {
            // Selling a call means the highest y can go is the strike price
            // the lowest y can go is the strike price + (ask - bid) * 100
            // if difference is negative then the option is in the money

            if (difference < 0) {
              profit_loss = difference * 100;
              y = Math.min(profit_loss, highest);
            } else {
              profit_loss = difference * 100;
              // set negative
              profit_loss = 0 - profit_loss;
            }
          }
        }
        data.push({ x: x, y: y });
      }

      return {
        name: name,
        data: data,
      };
    };

    const generateAnnotation = (option) => {
      // generate annotations for each options strike price
   
      var annotation;
      var strike_price = option.strike_price;
      
      annotation = {
        x: strike_price,
        strokeDashArray: 0,
        borderColor: "#775DD0",
        label: {
          borderColor: "#775DD0",
          style: {
            color: "#fff",
            background: "#775DD0",
          },
          text: option.type + " " + option.strike_price,
        },
      };

      return annotation;
    }

    var series = [];
    var annotations = [];

    for (var i = 0; i < this.optionsData.length; i++) {
      series.push(generateData(this.optionsData[i]));
      annotations.push(generateAnnotation(this.optionsData[i]));
    }

    return {
      series: series,
      chartOptions: {
        chart: {
          type: "area",
          height: 450,
        },
        annotations: {
              xaxis: annotations},
        dataLabels: {
          enabled: false,
        },
        stroke: {
          curve: "straight",
        },

        title: {
          text: "Area with Negative Values",
          align: "left",
          style: {
            fontSize: "14px",
          },
        },
        xaxis: {
          title: {
            text: "Stock Price",
          },
          type: "numeric",
          axisBorder: {
            show: true,
          },
          axisTicks: {
            show: true,
          },
        },
        yaxis: {
          title: {
            text: "Profit/Loss",
          },
          type: "numeric",
          tickAmount: 4,
          floating: false,

          labels: {
            style: {
              colors: "#8e8da4",
            },
            offsetY: 0,
            offsetX: 0,
          },
          axisBorder: {
            show: false,
          },
          axisTicks: {
            show: false,
          },
        },
        fill: {
          opacity: 0.5,
        },
        tooltip: {
          x: {
            format: "yyyy",
          },
          fixed: {
            enabled: false,
            position: "topRight",
          },
        },
        grid: {
          yaxis: {
            lines: {
              offsetX: 0,
              offsetY: 0,
            },
          },
          padding: {
            left: 20,
          },
        },
      },
    };
  },
};
</script>

<style scoped>
#chart {
  margin: 3em;
}
</style>
