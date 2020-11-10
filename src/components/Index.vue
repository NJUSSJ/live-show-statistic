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
              <span slot="title">板块热度</span>
            </el-menu-item>

            <el-menu-item index="2">
              <i class="el-icon-menu"></i>
              <span slot="title">弹幕词云</span>
            </el-menu-item>
          </el-menu>
        </el-aside>
        <el-container>
          <el-main>
            <div v-show="raceChart">
              <RaceChart/>
            </div>
            <div v-show="!raceChart">
            </div>
          </el-main>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>

<script>
import Header from '@/components/Header'
import RaceChart from '@/components/RaceChart'
export default {
  name: 'Index',
  components: {Header, RaceChart},
  data () {
    return {
      raceChart: true
    }
  },
  methods: {
    handleSelect (key, keyPath) {
      switch (key) {
        case '1':
          this.raceChart = true
          break
        case '2':
          this.raceChart = false
          break
      }
    }
  }
}
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
