<template>
  <div style="width: 100vw;height: 100vh;overflow: hidden;display: flex">
    <div style="flex: 1">
      <el-image style="width: 100%;height: 100%" fit="cover"
                src="./1.png"/>
    </div>
    <div class="welcome-title">
      <div style="font-size: 30px;font-weight: bold">欢迎光临红浪漫，请拿好您的手牌儿！</div>
      <div style="margin-top: 10px">发车喽，555</div>
    </div>
    <div style="width: 400px;background-color: aqua;z-index: 1">

      <router-view v-slot="{ Component }">
        <transition name="el-fade-in-linear" mode="out-in">
          <component :is="Component" style="height: 100%"/>
        </transition>
      </router-view>


    </div>


  </div>
</template>

<script setup>


</script>

<style scoped>
.welcome-title {
  position: absolute;
  bottom: 30px;
  left: 30px;
  color: aquamarine;
  text-shadow: 0 0 10px red;
}
</style>