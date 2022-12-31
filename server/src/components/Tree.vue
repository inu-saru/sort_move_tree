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
      },
      hovered: {
        element: null,
      },
      targetTreeChild: null,
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
      this.hoveringOnTree(event)      

      // sort
      const sortableNodes = Array.from(document.querySelectorAll('.node')).filter(node => node.style.display !== "none")
      if(sortableNodes && this.dragging) {
        const targetNode = sortableNodes.reduce((closestNode, sortableNode) => {
          const nodeLocation = sortableNode.getBoundingClientRect()
          const nodeLabelLocation = sortableNode.querySelector('a').getBoundingClientRect()
          const offsetY = event.pageY - (nodeLocation.top + nodeLabelLocation.height / 2)
          if(offsetY < 0 && offsetY > closestNode.offsetY) {
            return {
              offsetY: offsetY,
              element: sortableNode
            }
          } else {
            return closestNode
          }
        }, { offsetY: Number.NEGATIVE_INFINITY }).element

        this.placeHolder.element.remove()
        this.placeHolder.element = document.createElement("li")
        this.placeHolder.element.classList.add("place-holder")
        this.placeHolder.element.textContent = "▶︎"
        if (targetNode == undefined) {
          const rootTree = document.getElementById('tree').querySelector('ul')
          rootTree.appendChild(this.placeHolder.element)
        } else {
          this.placeHolder.element.style.textIndent = (targetNode.dataset.hierarchy * 16) + "px"
          targetNode.parentNode.insertBefore(this.placeHolder.element, targetNode)
        }
      }
    },
    createPlaceHolder() {
      this.placeHolder.element = document.createElement("li")
      this.placeHolder.element.classList.add("place-holder")
      this.placeHolder.element.appendChild(document.createTextNode("▶︎"))
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
    hoveringOnTree(event){
      const nodeLabels = Array.from(document.querySelectorAll('.nodeLabel'))
      const hoveredNodeLabel = nodeLabels.find((nodeLabel) => {
        const nodeLabelLocation = nodeLabel.getBoundingClientRect()
        return nodeLabelLocation.top <= event.pageY && event.pageY <= nodeLabelLocation.bottom
      })
      if(this.hovered.element) {
        this.hovered.element.classList.remove("hovered", "hoveredWithDrag")
      }
      if(hoveredNodeLabel && this.isInsindeTree(event)){
        if(this.dragging) {
          hoveredNodeLabel.classList.add("hoveredWithDrag")
          this.setTargetTreeChild(event) 
        } else {
          hoveredNodeLabel.classList.add("hovered")
        }
        this.hovered.element = hoveredNodeLabel
      }
    },
    isInsindeTree(event) {
      const treeLocation = document.getElementById('tree').getBoundingClientRect()
      return (treeLocation.top <= event.pageY && event.pageY <= treeLocation.bottom) && (treeLocation.left <= event.pageX && event.pageX <= treeLocation.right)
    },
    setTargetTreeChild(event) {
      const treeChildren = Array.from(document.querySelectorAll('.treeChild'))
      const targetTreeChild = treeChildren.reduce((shallowestTreeChild, treeChild) => {
        const treeChildLocation = treeChild.getBoundingClientRect()
        const depth = treeChildLocation.height
        if(shallowestTreeChild){
          if((treeChildLocation.top <= event.pageY && event.pageY <= treeChildLocation.bottom) && depth < shallowestTreeChild.depth ) {
            return {
              depth: depth,
              element: treeChild
            } 
          } else {
            return shallowestTreeChild
          }
        } else {
          return treeChild
        }
      }, { depth: Number.POSITIVE_INFINITY }).element
      this.targetTreeChild = targetTreeChild
    },
    mouseUp() {
      this.placeHolder.element.remove()
      this.draggingGhost.element.remove()
      this.element.style.display = "block"
      this.dragging = false
      this.targetTreeChild = null
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
