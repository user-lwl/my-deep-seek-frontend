<template>
  <router-view />
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}

body {
  background-image: url("./assets/beijing.jpg"); /* 设置背景图片的路径 */
  background-size: cover; /* 控制背景图片的大小 */
  /*background-color: #0a0909; !* 设置背景颜色 *!*/
}
</style>
