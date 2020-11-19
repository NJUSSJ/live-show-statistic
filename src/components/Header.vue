<template>
  <div class="header">
    <span class="logo">
      Live Show Dashboard
    </span>
    <el-button type="primary" id="start-btn">START</el-button>
    <el-button id="return button">Return</el-button>
  </div>
</template>
<script>
export default {
  name: 'Header',
  props: {
    showReturn: {
      type: Boolean,
      default: false
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

  #start-btn {
    margin-left: auto;
  }

</style>
