<template>
  <ul class="treeChild" :data-hierarchy="hierarchy">
    <li
      v-for="(list,index) in this.lists" 
      :key="index"
      :style="liHierarchy"
      :data-hierarchy="hierarchy"
      @mousedown="onMouseDown"
      class="node"
    >
      <a class="nodeLabel">
        {{ list.name }}
      </a>
      <Ul
        v-if="list.lists && list.lists.length"
        :lists="list.lists"
        :hierarchy="sonHierarchy"
        @onMouseDown="onMouseDown"
      />
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
  },
  methods: {
    onMouseDown(event) {
      this.$emit("onMouseDown", event)
    },
  },
}
</script>

<style scoped>
ul {
	list-style: none;
  padding: 0;
}
li {
  text-indent: var(--text-indent);
  width: 420px;
  background-color: #CCC;
  line-height: 32px;
  padding: 0;
  user-select: none;
}
a {
  pointer-events: none;
  user-select: none;
  display: block;
  box-sizing:border-box;
}
a:before {
	content: "▶︎";
	color: #000000;
	padding-right: 3px;
	line-height: 32px;
  margin-left: 10px;
}
</style>
