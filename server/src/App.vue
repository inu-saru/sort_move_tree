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
  display: block;
  height: 32px;
  width: 400px;
  display: block;
  margin: 0;
  line-height: 32px;
  padding: 0 10px;
  user-select: none;
  opacity: 0.5;
}
.hoveredPlaceHolder{
  background-color: bisque !important;
  opacity: 1;
}
.draggingGhost{
  width: 420px;
  background-color: #CCC;
  line-height: 32px;
  padding: 0;
  user-select: none;
  list-style: none;
  opacity: 0.7;
}
.draggingGhost:before {
	content: "▶︎";
	color: #000000;
	padding-right: 3px;
	line-height: 32px;
  margin-left: 10px;
}
.hovered {
  background-color: rgb(208, 252, 237);
}
.hoveredWithDrag {
  background-color:skyblue;
  border: 2px solid blue;
  height: 32px;
}
</style>
