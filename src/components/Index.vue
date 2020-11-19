<template>
  <div class="container">
    <el-container>
      <el-header height="60px">
        <Header />
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
              <RaceChart />
            </div>
            <div id="graphx-container" v-show="!Streaming"></div>
            <!-- <div v-show="!raceChart">
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
            </div> -->
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
export default {
  name: "Index",
  components: { Header, RaceChart, VueWordCloud },
  data() {
    return {
      Streaming: true,
      testData: [
                  ['起飞', 19],
                  ['芜湖', 3],
                  ['黄头发妹妹', 7],
                  ['再见', 3],
                ]
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
  },
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

#btn-container {
  width: 1000px;
  height: 10%;
  display: flex;
  align-items: center;
  margin: 0 auto;
}
</style>
