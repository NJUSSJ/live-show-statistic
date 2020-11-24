<template>
  <div id="graphx-container">
    <div id="echart" style="width: 1200px;height:700px; margin: 0 auto">

    </div>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  name: "GraphX",
  data () {
    return {

    }
  },
  methods: {
    async init () {
      // 基于准备好的dom，初始化echarts实例
      let myChart = this.$echarts.init(document.getElementById('echart'));

      //Graph data
      let graph = await this.getGraphData();

      // console.log(graph);
      
      let nodes = graph.nodes.map((node) => {
        return {
          id: node.id,
          value: node.rank,
          symbolSize: node.rank,
          itemStyle: null,
          category: node.category,
          name: node.name
        }
      })
      console.log(nodes);

      let edges = graph.edges.map((edge) => {
        return {
          id: edge.id,
          source: edge.source,
          target: edge.target
        }
      })
      
      console.log(edges)
      
      let categories = graph.categories.map((category) => {
        return {
          name: category
        }
      })

      console.log(categories)

      let option = {
        title: {
            top: 'bottom',
            left: 'right'
        },
        tooltip: {},
        legend: [{
            // selectedMode: 'single',
            data: categories.map(function (a) {
                return a.name;
            })
        }],
        animationDuration: 50000,
        animationEasingUpdate: 'quinticInOut',
        series : [
            {
                type: 'graph',
                layout: 'force',
                data: nodes,
                links: edges,
                categories: categories,
                roam: true,
                focusNodeAdjacency: true,
                itemStyle: {
                    borderColor: '#fff',
                    borderWidth: 1,
                    shadowBlur: 10,
                    shadowColor: 'rgba(0, 0, 0, 0.3)'
                },
                label: {
                    position: 'right',
                    formatter: '{b}'
                },
                lineStyle: {
                    color: 'source',
                    curveness: 0.3
                },
                emphasis: {
                    lineStyle: {
                        width: 10
                    }
                },
                force: {
                  edgeLength: [100,1000],//线的长度，这个距离也会受 repulsion，支持设置成数组表达边长的范围
                  repulsion: 1000//节点之间的斥力因子。值越大则斥力越大
                }
            }
        ]
      }
      myChart.setOption(option);
      // 使用刚指定的配置项和数据显示图表。
      // myChart.setOption(option);
    },
    async getGraphData () {
      let response = await fetch('http://127.0.0.1:5000/anchor')
      let json = await response.json()
      return json
    }
  },
  mounted () {
    this.init()
  }
}
</script>

<style scoped>
  #graphx-container {
    height: 100%;
    width: 100%;
    display: flex;
    justify-content: center;
  }
</style>>