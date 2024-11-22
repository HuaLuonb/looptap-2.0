<template>
  <div class="app">
    <h1>Loop Tap</h1>
    <input 
      v-model="developerCode" 
      placeholder="开发者代码" 
      type="password"
      @keyup.enter="checkDeveloperCode"
    >
    <button @click="checkDeveloperCode">进入</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      developerCode: ''
    }
  },
  methods: {
    checkDeveloperCode() {
      if (this.developerCode === '54188') {
        console.log('开发者模式')
      } else {
        alert('开发者代码错误')
      }
    }
  }
}
</script>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

input, button {
  margin: 10px;
  padding: 5px;
}
</style>
