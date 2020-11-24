<template>
  <div class="container">
    <el-container>
      <el-header height="60px">
        <Header @start='handleStart' @return='handleReturn' :startShow='startShow' :returnShow='returnShow' :startDiasble='startDiasble' :Streaming='Streaming'/>
      </el-header>
      <el-container>
        <el-aside width="200px">
          <el-menu
            default-active="1"
            class="el-menu-vertical-demo"
            @select="handleSelect"
          >
            <el-menu-item index="1">
              <i class="el-icon-location"></i>
              <span slot="title">Streaming</span>
            </el-menu-item>

            <el-menu-item index="2">
              <i class="el-icon-menu"></i>
              <span slot="title">GraphX</span>
            </el-menu-item>
          </el-menu>
        </el-aside>
        <el-container>
          <el-main>
            <div id="streaming-container" v-show="Streaming">
              <RaceChart :title="title" :current='current' v-show="current !== 'word-cloud'"/>
              <div v-show="current === 'word-cloud'">
                <vue-word-cloud
                  style="height: 480px; width: 100%"
                  :words="testData"
                  :color="
                    ([, weight]) =>
                      weight > 10
                        ? 'DeepPink'
                        : weight > 5
                        ? 'RoyalBlue'
                        : 'Indigo'
                  "
                  font-family="Roboto"
                />
              </div>
            </div>
            <div id="graphx-container" v-show="!Streaming">
              <GraphX/>
            </div>
            <!--  -->
          </el-main>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>

<script>
/* eslint-disable */
import Header from "@/components/Header";
import RaceChart from "@/components/RaceChart";
import VueWordCloud from "vuewordcloud";
import bus from "../event-bus"
import GraphX from "@/components/GraphX"
import * as moment from "moment";

export default {
  name: "Index",
  components: { Header, RaceChart, VueWordCloud, GraphX },

  data() {
    return {
      Streaming: true,
      testData: [
                  ['起飞', 19],
                  ['芜湖', 3],
                  ['黄头发妹妹', 7],
                  ['再见', 3],
                ],
      title: '当前直播板块热度排行',
      timeId: null,
      categortHotPointer: 0,
      hostHotPointer: 0,

      startDiasble: false,
      startShow: true,
      returnShow: false,

      current: 'recent-category',
      currentCategory: null,
      currentNickname: null
    };
  },

  methods: {
    handleSelect(key, keyPath) {
      switch (key) {
        case "1":
          this.Streaming = true;
          break;
        case "2":
          this.Streaming = false;
          break;
      }
    },
    async handleStart() {
      this.initWithOldData()
    },
    async initWithOldData() {
      if (this.timeId != null) {
        clearInterval(this.timeId)
      }

      bus.$emit('raceData', [])
      this.startDiasble = true;
      this.startShow = true;
      this.returnShow = true;
      this.title = '昨日 直播板块热度排行'
      this.current = 'category'

      let response = await fetch('http://127.0.0.1:5000/hot')
      let json = await response.json()

      this.categortHotPointer = 0
      this.timeId = setInterval(() => {
        let tmpData = []
        if (this.categortHotPointer >= json.length) clearInterval(this.timeId)
        let topList = json[this.categortHotPointer].list
        for (let j = 0; j < topList.length; j++) {
          let index = tmpData.findIndex(dd => dd.category === topList[j].name)
          if (index < 0) {
            let ss = {
              'category': topList[j].name,
              'nickname': topList[j].nickname,
              'currentTime': json[this.categortHotPointer].timeStamp
            }
            ss[json[this.categortHotPointer].timeStamp] = Number.isNaN(topList[j].hot) ? 10 : topList[j].hot
            tmpData.push(ss)
          } else {
            tmpData[index][json[this.categortHotPointer].timeStamp] = Number.isNaN(topList[j].hot) ? 10 : topList[j].hot
            tmpData[index].currentTime = json[this.categortHotPointer].timeStamp
          }
        }
        bus.$emit('raceData', tmpData)
        this.categortHotPointer ++
      }, 1000)
      // for (let i = 0; i < json.length; i++) {
      //   let tmpData = []
      //   let topList = json[i].list
        
      //   let result = await new Promise((resolve,reject) => {
      //     setTimeout(() => resolve(), 3000)
      //   })
      //   console.log(tmpData)
      //   bus.$emit('raceData', tmpData)
      // }
    },
    async initWithRecentData () {
      if (this.timeId != null) {
        clearInterval(this.timeId)
      }
      
      bus.$emit('raceData', [])
      
      this.current = 'recent-category'
      this.title = '当前 直播热度排行'
      this.startDiasble = false
      this.returnShow = false
      this.startShow = true

      let response = await fetch('http://127.0.0.1:5000/hot?latest=1')
      let json = await response.json()

      let list = json.list

      let tmpData = []
      let currentTime = moment().format('h:mm')
      console.log(currentTime)
      for (let i = 0; i < list.length; i++) {
        let index = tmpData.findIndex(dd => dd.category === list.name)
          if (index < 0) {
            let ss = {
              'category': list[i].name,
              'nickname': list[i].nickname,
              'currentTime': currentTime
            }
            ss[currentTime] = Number.isNaN(list[i].hot) ? 10 : list[i].hot
            tmpData.push(ss)
          }
      }

      bus.$emit('raceData', tmpData)
    },
    async handleCategoryRaceData(category, nickname) {
      if (this.timeId != null) {
        clearInterval(this.timeId)
      }
      bus.$emit('raceData', [])

      this.returnShow = true
      this.title = nickname + " 昨日主播热度排行"
      this.current = 'host'
      this.currentCategory = category

      let response = await fetch('http://127.0.0.1:5000/hot/'+category)
      let json = await response.json()

      this.hostHotPointer = 0
      this.timeId = setInterval(() => {
        let tmpData = []
        if (this.hostHotPointer >= json.length) clearInterval(this.timeId)
        let topList = json[this.hostHotPointer].list
        for (let j = 0; j < topList.length; j++) {
          let index = tmpData.findIndex(dd => dd.category === topList[j].host)
          if (index < 0) {
            let ss = {
              'category': topList[j].host,
              'nickname': topList[j].host,
              'currentTime': json[this.hostHotPointer].time
            }
            ss[json[this.hostHotPointer].time] = Number.isNaN(topList[j].hot) ? 10 : topList[j].hot
            tmpData.push(ss)
          } else {
            tmpData[index][json[this.hostHotPointerr].time] = Number.isNaN(topList[j].hot) ? 10 : topList[j].hot
            tmpData[index].currentTime = json[this.hostHotPointer].time
          }
        }
        bus.$emit('raceData', tmpData)
        this.hostHotPointer ++
      }, 1000)
    },
    async handleRecentCategoryData(category, nickname) {
      if (this.timeId != null) {
        clearInterval(this.timeId)
      } 
      bus.$emit('raceData', [])

      this.returnShow = true
      this.startShow = false
      this.title = nickname + " 当前主播热度排行"
      this.current = 'recent-host'
      this.currentCategory = category
      this.currentNickname = nickname

      let response = await fetch('http://127.0.0.1:5000/hot/' + category + '?latest=1')
      let json = await response.json()


      let list = json.list

      let tmpData = []
      let currentTime = moment().format('h:mm')
      for (let i = 0; i < list.length; i++) {
        let index = tmpData.findIndex(dd => dd.category === list.host)
          if (index < 0) {
            let ss = {
              'category': list[i].host,
              'nickname': list[i].host,
              'currentTime': currentTime
            }
            ss[currentTime] = Number.isNaN(list[i].hot) ? 10 : list[i].hot
            tmpData.push(ss)
          }
      }

      bus.$emit('raceData', tmpData)

    },
    async handleHostWorlCloud(host) {
      console.log(host)
      this.current = 'word-cloud'
    },
    handleReturn () {
      if (this.current === 'word-cloud') {
        this.handleRecentCategoryData(this.currentCategory, this.currentNickname)
      } else if (this.current === 'recent-host') {
        this.initWithRecentData()
      } else if (this.current === 'host') {
        this.handleStart()
      } else if (this.current === 'category') {
        this.initWithRecentData()
      }
    },
  },

  created() {
    bus.$on('categoryRaceData', (category, nickname) => {
      this.handleCategoryRaceData(category, nickname)
    })
    bus.$on('hostWordCloud', (host) => {
      this.handleHostWorlCloud(host)
    })
    bus.$on('recentCategoryRaceData', (category, nickname) => {
      this.handleRecentCategoryData(category, nickname)
    })
  },

  mounted() {
    this.initWithRecentData()
  }
};
</script>

<style scope>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  height: 100vh;
}

.el-container {
  width: 100%;
}

.el-header {
  background-color: #ffffff;
  color: #333;
  text-align: center;
  z-index: 1;
  border-bottom: 1px solid #dbdbdb;
}

.el-aside {
  background-color: #ffffff;
  color: #333;
  text-align: center;
}

.el-main {
  background-color: #e9eef3;
  color: #333;
  text-align: center;
}

.el-menu {
  height: 100%;
}

</style>
