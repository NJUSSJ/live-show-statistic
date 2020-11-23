<template>
  <div>
    <div id="title-container">
      <span id="title">{{this.title}}</span>
    </div>
    <div id="chart-container" v-loading="isLoading" element-loading-background="rgba(0, 0, 0, 0)">
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
  </div>
</template>

<script>
/* eslint-disable */
import * as d3 from "d3";
import * as moment from "moment";
import bus from '../event-bus'
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
      default: 800,
    },
    id: {
      type: Number,
      default: 1
    },
    title: {
      type: String,
      default: '直播板块热度排行'
    },
    raceData: {
      type: Array,
      default: () => []
    },
    current: {
      type: String,
      default: 'category'
    }  
  },
  data() {
    return {
      width: 1000,
      height: 600,
      margin: { left: 120, right: 90, top: 40, bottom: 0 },
      curDate: null,
      lastDate: null,
      yearLabel: "",
      dateLabel: "",
      timer: null,
      dataList: [],
      g: null,
      topAxis: null,
      isLoading: false
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
  created () {
    bus.$on('raceData', (raceData) => {
      if (raceData.length === 0) {
        if (this.g != null) {
          this.g.selectAll("g.bar.active").remove()
        }
        this.isLoading = true
      } else {
        this.isLoading = false
        this.dataList = raceData
        console.log(this.dataList)
        this.renderMockData()
      }
      
    })
  },
  methods: {
    renderMockData() {
      if (this.g == null) {
        this.g = d3.select("#race-bar");
      }
      const g = this.g;
      if (this.topAxis == null) {
        this.topAxis = g.append("g").attr("transform", `translate(50, 0)`);
      }
      const topAxis = this.topAxis;

      let curDate = this.dataList[0]['currentTime'];

      let tickData = this.dataList
        .sort((a, b) => d3.descending(+a[curDate], +b[curDate]))
        .slice(0, this.topN);

      /* Set scale and coordinate axis --start */
      const xScale = d3.scaleLinear(
        [0, d3.max(tickData, (d) => d[curDate])],
        [0, this.innerWidth]
      );

      // console.log(xScale)

      // const xScale = (num, width = this.innerWidth) => {
      //   let result = num / d3.max(tickData, (d) => d[curDate]) * width;
      //   if (Number.isNaN(result)) {
      //     console.log(num + ' ' + d3.max(tickData, (d) => d[curDate]) + '!!!!!!')
      //   }
      //   return result;
      // }
      // const xScale = (d) => d / (d3.max(tickData, (d) => d[curDate])) * this.innerWidth
      
      const yScale = d3
        .scaleBand(
          tickData.map((it) => it["category"]),
          [0, this.innerHeight]
        )
        .padding(0.2);

      const xAxis = d3.axisTop(xScale).ticks(4);
      /* Set scale and coordinate axis --end */
      const dataG = g
        .selectAll("g.bar.active")
        .data(tickData, (d) => d["category"]);
      if (dataG.size()) {
        // new bar
        const newG = dataG.enter().append("g").attr("class", "bar active");
        newG
          .append("rect")
          .on("click", (e, d) => this.handleClick(d['category'], d['nickname']))
          .attr("cursor", "pointer")
          .attr("width", (d) => xScale(d[curDate]))
          .attr("height", (d) => yScale.bandwidth())
          .attr("y", this.innerHeight)
          .attr("x", 50)
          .attr("fill", (d) => this.colorScale(d["category"]))
          .attr("fill-opacity", 0.6)
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("y", (d) => yScale(d["category"]));
        newG
          .append("text")
          .text((d) => d3.format(",")(d[curDate]))
          .attr("class", "num")
          .attr("y", (d) => this.innerHeight)
          .attr("x", (d) => (xScale(d[curDate]) + 62))
          .attr("dy", (d) => yScale.bandwidth() / 2 + 15)
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("y", (d) => yScale(d["category"]));

        newG
          .append("text")
          .text((d) => d["nickname"])
          .attr("class", "label")
          .attr("font-weight", "bold")
          .attr("y", this.innerHeight)
          .attr("x", (d) => (xScale(d[curDate]) + 62))
          .attr("dy", (d) => yScale.bandwidth() / 2 - 2)
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("y", (d) => yScale(d["category"]));

        // upate bar
        dataG
          .select("rect")
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("width", (d) => xScale(d[curDate]))
          .attr("y", (d) => yScale(d["category"]));

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
          .attr("y", (d) => yScale(d["category"]))
          .attr("x", (d) => (xScale(+d[curDate]) + 62));

        dataG
          .select("text.label")
          .transition()
          .duration(this.interval)
          .ease(d3.easeLinear)
          .attr("x", (d) => (xScale(+d[curDate]) + 62))
          .attr("y", (d) => yScale(d["category"]));

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
            return this.colorScale(d["category"]);
          })
          .attr("cursor", "pointer")
          .attr("fill-opacity", 0.6)
          .attr("width", (d) => xScale(+d[curDate]))
          .attr("height", (d) => yScale.bandwidth())
          .attr("y", (d) => yScale(d["category"]))
          .attr("x", 50)
          .on("click", (e, d) => this.handleClick(d["category"], d['nickname']));

        res
          .append("text")
          .text((d) => d3.format(",")(d[curDate]))
          .attr("class", "num")
          .attr("y", (d) => yScale(d["category"]))
          .attr("x", (d) => (xScale(+d[curDate]) + 62))
          .attr("dy", (d) => yScale.bandwidth() / 2 + 15);

        res
          .append("text")
          .text((d) => d["nickname"])
          .attr("class", "label")
          .attr("font-weight", "bold")
          .attr("y", (d) => yScale(d["category"]))
          .attr("x", (d) => (xScale(+d[curDate]) + 62))
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
    handleClick(category, nickname) {
      if (this.current === 'category') {
        bus.$emit('categoryRaceData', category, nickname);
      } else if (this.current === 'recent-category') {
        bus.$emit('recentCategoryRaceData', category, nickname);
      } else if (this.current === 'recent-host'){
        bus.$emit('hostWordCloud', category, nickname);
      }
    }
  },
  watch: {
    curDate(val) {
      if (val < '12:00') {
        this.yearLabel = moment().format("MM-DD");
      } else {
        this.yearLabel = this.current === 'recent-category' || this.current === 'recent-host'? moment().format("MM-DD") : moment().subtract(1, 'days').format("MM-DD");
      }
      this.dateLabel = val;
    }
  },
  sockets: {
    connect() {
      console.log("connect");
    },
    disconnect() {
      console.log("disconnect");
    }
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

#title {
  cursor: pointer;
  font-weight: bold;
  font-size: 18px;
  float: left;
  margin-left: 5%;
}

#title-container {
  margin: 0 auto;
  width: 1000px;
  display: flex;
  align-items: center;
}
</style>