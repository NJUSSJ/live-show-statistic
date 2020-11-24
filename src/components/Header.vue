<template>
  <div class="header">
    <span class="logo">
      Live Show Dashboard
    </span>
    <div id="btn-container" v-show="Streaming">
      <el-button type="primary" id="start-btn" @click="start" :disabled='startDiasble' v-show="startShow">START</el-button>
      <el-button :id="startShow ? 'return-btn': 'return-btn-no-start'" v-show="returnShow" @click="back">Return</el-button>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Header',
  props: {
    startShow: {
      type: Boolean,
      default: true
    },
    returnShow: {
      type: Boolean
    },
    startDiasble: {
      type: Boolean
    },
    Streaming: {
      type: Boolean
    }
  },
  data () {
    return {
    }
  },
  methods: {
    start () {
      this.$emit('start')
    },
    back () {
      this.$emit('return')
    }
  }
}
</script>

<style scoped>
  .header {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    box-sizing: border-box;
  }

  .logo {
    font-size: 16px;
    font-weight: bold;
  }

  #btn-container {
    margin-left: auto;
    display: flex;
    height: 70%;
  }

  #start-btn {
    margin-left: auto;
  }

  #return-btn {
  }

  #return-btn-no-start {
    margin-left: auto;
  }

</style>
