<template>
  <div id="app">
    <Tree />
  </div>
</template>

<script>
import Tree from './components/Tree.vue'

export default {
  name: 'App',
  components: {
    Tree
  }
}
</script>

<style>
.place-holder{
  color: brown;
  display: block;
  height: 32px;
  width: 400px;
  display: block;
  margin: 0;
  line-height: 32px;
  padding: 0 10px;
}
</style>
