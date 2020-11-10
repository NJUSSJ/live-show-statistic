<template>
  <div>
    <svg :width="width" :height="height">
      <g transform="translate(0, 40)" id="race-bar">
        <text :x="innerWidth - 90" :y="innerHeight - 140" class="dateLabel">
          {{ dateLabel }}
        </text>
        <text :x="innerWidth - 100" :y="innerHeight - 50" class="yearLabel">
          {{ yearLabel }}
        </text>
      </g>
    </svg>
  </div>
</template>

<script>
/* eslint-disable */
import * as d3 from "d3";
import * as moment from "moment";
export default {
  name: "RaceChart",
  props: {
    csvUrl: {
      type: String,
      default: "https://covid.ourworldindata.org/data/ecdc/total_cases.csv",
    },
    topN: {
      type: Number,
      default: 10,
    },
    interval: {
      type: Number,
      default: 500,
    },
    excludeChina: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      isLoading: false,
      width: 1000,
      height: 700,
      margin: { left: 120, right: 90, top: 40, bottom: 0 },
      curDate: null,
      lastDate: null,
      yearLabel: "",
      dateLabel: "",
      timer: null,
      dataList: [],
      g: null,
      topAxis: null
    };
  },
  computed: {
    innerHeight() {
      return this.height - this.margin.top;
    },
    innerWidth() {
      return this.width - this.margin.left - this.margin.right;
    },
    colorScale() {
      return d3.scaleOrdinal(d3.schemeCategory10);
    },
  },
  mounted() {
    // this.init();
    this.$socket.emit("connect");
  },
  methods: {
    async init() {
      this.isLoading = true;
      this.dataList = await this.getDataFromCsv(this.csvUrl);
      console.log(this.dataList);
      this.renderData();
      this.isLoading = false;
    },
    getDataFromCsv(url) {
      return new Promise((resolve, reject) => {
        d3.csv(url)
          .then((d) => {
            const china = this.excludeChina ? "China" : "Something Else";
            const data = [];

            this.lastDate = d[d.length - 1].date;
            d.forEach((ob) => {
              let date = null;
              for (let k in ob) {
                if (k === "date") {
                  date = ob[k];
                } else {
                  if (
                    k.trim() === "World" ||
                    k.trim() === china ||
                    k.trim() === "International"
                  ) {
                    //pass
                  } else {
                    const idx = data.findIndex((dd) => dd.country === k);
                    if (idx < 0) {
                      const ss = {};
                      ss.country = k;
                      ss[date] = ob[k];
                      data.push(ss);
                    } else {
                      data[idx][date] = ob[k];
                    }
                  }
                }
              }
            });
            resolve(data);
          })
          .catch((e) => {
            reject(e);
          });
      });
    },
    renderData() {
      if (this.timer) this.timer.stop();
      const g = d3.select("#race-bar");
      const topAxis = g.append("g").attr("transform", `translate(50, 0)`);

      let curDate = this.curDate;
      this.timer = d3.interval(() => {
        let tickData = this.dataList
          .sort((a, b) => {
            return d3.descending(+a[curDate], +b[curDate]);
          })
          .slice(0, this.topN);
        /* Set scale and coordinate axis --start */
        const xScale = d3.scaleLinear(
          [0, d3.max(tickData, (d) => +d[curDate])],
          [0, this.innerWidth]
        );
        const yScale = d3
          .scaleBand(
            tickData.map((it) => it["country"]),
            [0, this.innerHeight]
          )
          .padding(0.2);

        const xAxis = d3.axisTop(xScale).ticks(4);

        /* Set scale and coordinate axis --end */

        const dataG = g
          .selectAll("g.bar.active")
          .data(tickData, (d) => d["country"]);
        if (dataG.size()) {
          // new bar
          const newG = dataG.enter().append("g").attr("class", "bar active");
          newG
            .append("rect")
            .on("click", (e, d) => console.log(d["country"]))
            .attr("cursor", "pointer")
            .attr("width", (d) => xScale(d[curDate]))
            .attr("height", (d) => yScale.bandwidth())
            .attr("y", this.innerHeight)
            .attr("x", 50)
            .attr("fill", (d) => this.colorScale(d["country"]))
            .attr("fill-opacity", 0.6)
            .transition()
            .duration(this.interval)
            .ease(d3.easeLinear)
            .attr("y", (d) => yScale(d["country"]));
          newG
            .append("text")
            .text((d) => d3.format(",")(d[curDate]))
            .attr("class", "num")
            .attr("y", (d) => this.innerHeight)
            .attr("x", (d) => xScale(d[curDate]) + 62)
            .attr("dy", (d) => yScale.bandwidth() / 2 + 15)
            .transition()
            .duration(this.interval)
            .ease(d3.easeLinear)
            .attr("y", (d) => yScale(d["country"]));

          newG
            .append("text")
            .text((d) => d["country"])
            .attr("class", "label")
            .attr("font-weight", "bold")
            .attr("y", this.innerHeight)
            .attr("x", (d) => xScale(d[curDate]) + 62)
            .attr("dy", (d) => yScale.bandwidth() / 2 - 2)
            .transition()
            .duration(this.interval)
            .ease(d3.easeLinear)
            .attr("y", (d) => yScale(d["country"]));

          // upate bar
          dataG
            .select("rect")
            .transition()
            .duration(this.interval)
            .ease(d3.easeLinear)
            .attr("width", (d) => xScale(d[curDate]))
            .attr("y", (d) => yScale(d["country"]));

          dataG
            .select("text.num")
            .transition()
            .duration(this.interval)
            .ease(d3.easeLinear)
            .tween("text", function (d) {
              const prev = d3.select(this).text().replace(/,/g, "");
              const i = d3.interpolateRound(prev, +d[curDate]);
              return function (t) {
                d3.select(this).text(d3.format(",")(i(t)));
              };
            })
            .attr("y", (d) => yScale(d["country"]))
            .attr("x", (d) => xScale(d[curDate]) + 62);

          dataG
            .select("text.label")
            .transition()
            .duration(this.interval)
            .ease(d3.easeLinear)
            .attr("x", (d) => xScale(d[curDate]) + 62)
            .attr("y", (d) => yScale(d["country"]));

          // delete bar
          dataG
            .exit()
            .attr("class", "bar")
            .transition()
            .duration(this.interval)
            .ease(d3.easeLinear)
            .attr("transform", `translate(0, ${this.innerHeight})`)
            .remove();

          topAxis
            .transition()
            .duration(this.interval)
            .call(xAxis)
            .selectAll("line")
            .attr("y2", this.innerHeight)
            .attr("stroke", "white")
            .attr("y1", -6);
        } else {
          const res = dataG.enter().append("g").attr("class", "bar active");

          res
            .append("rect")
            .attr("fill", (d, i) => {
              return this.colorScale(d["country"]);
            })
            .attr("cursor", "pointer")
            .attr("fill-opacity", 0.6)
            .attr("width", (d) => xScale(d[curDate]))
            .attr("height", (d) => yScale.bandwidth())
            .attr("y", (d) => yScale(d["country"]))
            .attr("x", 50)
            .on("click", (e, d) => console.log(d["country"]));

          res
            .append("text")
            .text((d) => d3.format(",")(d[curDate]))
            .attr("class", "num")
            .attr("y", (d) => yScale(d["country"]))
            .attr("x", (d) => xScale(d[curDate]) + 62)
            .attr("dy", (d) => yScale.bandwidth() / 2 + 15);

          res
            .append("text")
            .text((d) => d["country"])
            .attr("class", "label")
            .attr("font-weight", "bold")
            .attr("y", (d) => yScale(d["country"]))
            .attr("x", (d) => xScale(d[curDate]) + 62)
            .attr("dy", (d) => yScale.bandwidth() / 2 - 2);

          topAxis.call(xAxis);
          topAxis
            .selectAll("line")
            .attr("y2", innerHeight)
            .attr("stroke", "white");
          topAxis.selectAll("line").attr("y1", -6);
        }

        const tomorrow = moment(curDate).add(1, "days");
        const dateEnd = moment(this.lastDate);
        if (tomorrow > dateEnd) {
          this.timer.stop();
        } else {
          curDate = tomorrow.format("YYYY-MM-DD");
          this.curDate = curDate;
        }
      }, this.interval);
    },
    renderMockData() {
      if (this.g == null) {
        this.g = d3.select("#race-bar");
      }
      const g = this.g;
      if (this.topAxis == null) {
        this.topAxis = g.append("g").attr("transform", `translate(50, 0)`);
      }
      const topAxis = this.topAxis;

      let curDate = this.dataList[0]['currentDate'];
      this.curDate = curDate;

      let tickData = this.dataList
        .sort((a, b) => d3.descending(+a[curDate], +b[curDate]))
        .slice(0, this.topN);

      /* Set scale and coordinate axis --start */
      const xScale = d3.scaleLinear(
        [0, d3.max(tickData, (d) => +d[curDate])],
        [0, this.innerWidth]
      );
      const yScale = d3
        .scaleBand(
          tickData.map((it) => it["country"]),
          [0, this.innerHeight]
        )
        .padding(0.2);

      const xAxis = d3.axisTop(xScale).ticks(4);
      /* Set scale and coordinate axis --end */
      const dataG = g
        .selectAll("g.bar.active")
        .data(tickData, (d) => d["country"]);
      if (dataG.size()) {
        // new bar
        const newG = dataG.enter().append("g").attr("class", "bar active");
        newG
          .append("rect")
          .on("click", (e, d) => console.log(d["country"]))
          .attr("cursor", "pointer")
          .attr("width", (d) => xScale(d[curDate]))
          .attr("height", (d) => yScale.bandwidth())
          .attr("y", this.innerHeight)
          .attr("x", 50)
          .attr("fill", (d) => this.colorScale(d["country"]))
          .attr("fill-opacity", 0.6)
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("y", (d) => yScale(d["country"]));
        newG
          .append("text")
          .text((d) => d3.format(",")(d[curDate]))
          .attr("class", "num")
          .attr("y", (d) => this.innerHeight)
          .attr("x", (d) => xScale(d[curDate]) + 62)
          .attr("dy", (d) => yScale.bandwidth() / 2 + 15)
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("y", (d) => yScale(d["country"]));

        newG
          .append("text")
          .text((d) => d["country"])
          .attr("class", "label")
          .attr("font-weight", "bold")
          .attr("y", this.innerHeight)
          .attr("x", (d) => xScale(d[curDate]) + 62)
          .attr("dy", (d) => yScale.bandwidth() / 2 - 2)
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("y", (d) => yScale(d["country"]));

        // upate bar
        dataG
          .select("rect")
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("width", (d) => xScale(d[curDate]))
          .attr("y", (d) => yScale(d["country"]));

        dataG
          .select("text.num")
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .tween("text", function (d) {
            const prev = d3.select(this).text().replace(/,/g, "");
            const i = d3.interpolateRound(prev, +d[curDate]);
            return function (t) {
              d3.select(this).text(d3.format(",")(i(t)));
            };
          })
          .attr("y", (d) => yScale(d["country"]))
          .attr("x", (d) => xScale(d[curDate]) + 62);

        dataG
          .select("text.label")
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("x", (d) => xScale(d[curDate]) + 62)
          .attr("y", (d) => yScale(d["country"]));

        // delete bar
        dataG
          .exit()
          .attr("class", "bar")
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("transform", `translate(0, ${this.innerHeight})`)
          .remove();

        topAxis
          .transition()
          .duration(this.interval)
          .call(xAxis)
          .selectAll("line")
          .attr("y2", this.innerHeight)
          .attr("stroke", "white")
          .attr("y1", -6);
      } else {
        const res = dataG.enter().append("g").attr("class", "bar active");

        res
          .append("rect")
          .attr("fill", (d, i) => {
            return this.colorScale(d["country"]);
          })
          .attr("cursor", "pointer")
          .attr("fill-opacity", 0.6)
          .attr("width", (d) => xScale(d[curDate]))
          .attr("height", (d) => yScale.bandwidth())
          .attr("y", (d) => yScale(d["country"]))
          .attr("x", 50)
          .on("click", (e, d) => console.log(d["country"]));

        res
          .append("text")
          .text((d) => d3.format(",")(d[curDate]))
          .attr("class", "num")
          .attr("y", (d) => yScale(d["country"]))
          .attr("x", (d) => xScale(d[curDate]) + 62)
          .attr("dy", (d) => yScale.bandwidth() / 2 + 15);

        res
          .append("text")
          .text((d) => d["country"])
          .attr("class", "label")
          .attr("font-weight", "bold")
          .attr("y", (d) => yScale(d["country"]))
          .attr("x", (d) => xScale(d[curDate]) + 62)
          .attr("dy", (d) => yScale.bandwidth() / 2 - 2);

        topAxis.call(xAxis);
        topAxis
          .selectAll("line")
          .attr("y2", innerHeight)
          .attr("stroke", "white");
        topAxis.selectAll("line").attr("y1", -6);
      }

      this.curDate = curDate;
    },
  },
  watch: {
    curDate(val) {
      this.yearLabel = moment(val).format("YYYY");
      this.dateLabel = moment(val).format("DD MMM");
    },
  },
  sockets: {
    newData(data) {
      console.log(data);
      this.dataList = data;
      this.renderMockData();
    },
    connect() {
      console.log("connect");
      this.$socket.emit("getData");
    },
    disconnect() {
      console.log("disconnect");
    },
  },
};
</script>

<style scoped>
.bar {
  background: blue;
  height: 60px;
  color: #fff;
  margin-bottom: 10px;
}

.a {
  color: red;
}

.update {
  color: black;
  position: relative;
}

.enter {
  color: green;
  position: relative;
}

.yearLabel {
  font-size: 80px;
  font-weight: bold;
  fill: #ccc;
}
.dateLabel {
  font-size: 50px;
  font-weight: bold;
  fill: #ccc;
}
.label {
  text-align: right;
}
</style>