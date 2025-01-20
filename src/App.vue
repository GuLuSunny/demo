<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
html,
body,
#app {
  padding: 0;
  margin: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

ol,
ul,
li {
  list-style: none;
}


@font-face {
  font-family: 'PangMenZhengDao';
  src: url('./assets/font/PangMenZhengDao-nhX5wd4I.woff') format('woff'),
}

@font-face {
  font-family: 'PuHuiTi';
  src: url('./assets/font/PuHuiTi-Bt4JbhD9.woff') format('woff'),
}

@font-face {
  font-family: 'YouShe';
  src: url('./assets/font/YouShe-fahbdFno.woff') format('woff'),
}
</style>
