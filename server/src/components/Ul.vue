<template>
  <ul>
    <li
      v-for="(list,index) in this.lists" 
      :key="index"
      :style="liHierarchy"
    >
      <a>{{ list.name }}</a>
      <Ul :lists="list.lists" :hierarchy="sonHierarchy" />
    </li>
  </ul>
</template>

<script>
import Ul from './Ul.vue'

export default {
  name: 'Ul',
  components: {
    Ul
  },
  props: {
    lists: {
      type: Array,
    },
    hierarchy: {
      type: Number,
      default: 0
    }
  },
  computed: {
    liHierarchy() {
      return {
        '--text-indent': (this.hierarchy * 16) + "px"
      }
    },
    sonHierarchy() {
      return this.hierarchy + 1
    }
  }
}
</script>

<style scoped>
ul {
	list-style: none;
  padding: 0;
}
li {
  text-indent: var(--text-indent);
}
a {
  height: 32px;
  display: block;
  margin: 0;
  line-height: 32px;
  padding: 0 10px;
}
a:hover {
  background-color: #dfdfdf;
}
a:before {
	content: "▶︎";
	color: #000000;
	padding-right: 3px;
	line-height: 32px;
}
</style>
