<template>
  <div id="app">
    <router-view/>
  </div>
</template>
<script>
/**
 * dom 加载完毕之后 设置字体大小 (rem 的值相当于根元素 font-size 值的倍数.
 *  1rem = 根元素font-size ,2rem = 根元素font-size * 2)
 * 设置 viewport 是为了解决禁止程序在移动端的自行缩放,影响体验
 */
document.addEventListener('DOMContentLoaded',()=>{
  const html = document.querySelector('html')
  let fontSize = window.innerWidth / 10
  fontSize = fontSize > 50 ? 50 : fontSize
  html.style.fontSize = fontSize + 'px'

  const viewport = document.querySelector('meta[name=viewport]')
  viewport.setAttribute('content','width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=no')
})
</script>
<style lang="scss">
</style>
