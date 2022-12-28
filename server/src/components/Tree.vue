<template>
  <div id="tree">
    <Ul
      :lists="this.lists"
      @onMouseDown="mouseDown"
    />
    <div ref="draggingGhost" />
  </div>
</template>

<script>
import Ul from './Ul.vue'

export default {
  components: {
    Ul
  },
  data(){
    return {
      lists: [
        {
          id: 1,
          name: '親フォルダA',
          lists: [
            {
              id: 11,
              name: '子フォルダAA',
              lists: [
                {
                  id: 111,
                  name: '孫フォルダAAA',
                  lists: []
                },
                {
                  id: 112,
                  name: '孫フォルダAAB',
                  lists: []
                },
              ]
            },
            {
              id: 12,
              name: '子フォルダAB',
              lists: []
            }
          ]
        },
        {
          id: 2,
          name: '親フォルダB',
          lists: [
            {
              id: 21,
              name: '子フォルダBA',
              lists: []
            }
          ]
        },
        {
          id: 3,
          name: '親フォルダC',
          lists: [
  
          ]
        },
        {
          id: 4,
          name: '親フォルダD',
          lists: [
            {
              id: 41,
              name: '子フォルダDA',
              lists: []
            }
          ]
        },
      ],
      element: '',
      dragging: false,
      pageX:0,
      pageY:0,
      top: 0,
      left: 0,
      placeHolder: {
        element: '',
        isFirst: true
      },
      draggingGhost: {
        element: '',
      }
    }
  },
  methods:{
    mouseDown(event){
      this.dragging = true
      this.element = event.target
      this.pageX = event.pageX
      this.pageY = event.pageY
      this.top = this.element.getBoundingClientRect().top
      this.left = this.element.getBoundingClientRect().left
      this.placeHolder.isFirst = true
    },
    mouseMove(event){
      if(this.dragging){
        if(this.placeHolder.isFirst){
          this.createPlaceHolder()
          this.createDraggingGhost()
          this.element.style.display = "none"
          this.placeHolder.isFirst = false;
        }
        this.moveDraggingGhost(event)
      }
    },
    createPlaceHolder() {
      this.placeHolder.element = document.createElement("a")
      this.placeHolder.element.classList.add("place-holder")
      this.element.parentNode.insertBefore(this.placeHolder.element, this.element.nextSibling)
    },
    createDraggingGhost() {
      this.draggingGhost.element = document.createElement("li")
      this.draggingGhost.element.classList.add("draggingGhost")
      this.draggingGhost.element.appendChild(document.createTextNode(this.element.children[0].textContent))
      this.draggingGhost.element.style.position = "absolute"
      this.$refs.draggingGhost.appendChild(this.draggingGhost.element)
      this.draggingGhost.element.style.top = `${this.top}px`
      this.draggingGhost.element.style.left = `${this.left}px`
    },
    moveDraggingGhost(event) {
      const moveX = event.pageX - this.pageX
      const moveY = event.pageY - this.pageY
      this.draggingGhost.element.style.top = `${this.top + moveY}px`
      this.draggingGhost.element.style.left = `${this.left + moveX}px`
    },
    mouseUp() {
      this.placeHolder.element.remove()
      this.draggingGhost.element.remove()
      this.element.style.display = "block"
      this.dragging = false
    }
  },
  mounted() {
    window.addEventListener('mousemove', this.mouseMove)
    window.addEventListener('mouseup', this.mouseUp)
  },
  beforeunMount(){
    window.removeEventListener('mousemove', this.mouseMove)
    window.removeEventListener('mouseup', this.mouseUp)
  }
}
</script>

<style scoped>
#tree {
  width: 420px;
  background-color: #CCC;
}
</style>
