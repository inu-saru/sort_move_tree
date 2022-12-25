<template>
  <div id="tree">
    <Ul
      :lists="this.lists"
      @onMouseDown="mouseDown"
    />
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
            {
              id: 31,
              name: '子フォルダCA',
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
      }
    }
  },
  methods:{
    mouseDown(event){
      this.dragging = true
      this.element = event.target
      this.element.style.position = "absolute"
      this.pageX = event.pageX
      this.pageY = event.pageY
      this.top = this.element.getBoundingClientRect().top
      this.left = this.element.getBoundingClientRect().left
      this.placeHolder.isFirst = true
    },
    mouseMove(event){
      if(this.dragging){
        if(this.placeHolder.isFirst){
          this.placeHolder.element = document.createElement("a")
          this.placeHolder.element.classList.add("place-holder")
          this.placeHolder.element.appendChild(document.createTextNode("place-holder"))
          this.element.parentNode.insertBefore(this.placeHolder.element, this.element.nextSibling);
          this.placeHolder.isFirst = false;
        }
        const moveX = event.pageX - this.pageX;
        const moveY = event.pageY - this.pageY;
        this.element.style.top = `${this.top + moveY}px`
        this.element.style.left = `${this.left + moveX}px`
      }
    },
    mouseUp() {
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
