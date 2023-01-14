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
      dragging: false,
      isDraggedOverOriginalLocation: false,
      dragged: {
        element: null,
        pageX:0,
        pageY:0,
        top: 0,
        left: 0,
      },
      placeHolder: {
        element: null,
        isFirst: true,
      },
      draggingGhost: {
        element: null,
      },
      hovered: {
        element: null,
      },
      hoveredTreeChild: {
        element: null,
      },
      belowNode: {
        element: null,
      },
      aboveNode: {
        element: null,
      }
    }
  },
  methods:{
    mouseDown(event){
      this.dragging = true
      this.dragged.element = event.target
      this.dragged.pageX = event.pageX
      this.dragged.pageY = event.pageY
      this.dragged.top = this.dragged.element.getBoundingClientRect().top
      this.dragged.left = this.dragged.element.getBoundingClientRect().left
      this.placeHolder.isFirst = true
    },
    mouseMove(event){
      if(this.dragging){
        if(this.placeHolder.isFirst){
          this.setPlaceHolder(this.dragged.element.dataset.hierarchy)
          this.dragged.element.parentNode.insertBefore(this.placeHolder.element, this.dragged.element.nextSibling)
          this.createDraggingGhost()
          this.dragged.element.style.display = "none"
          this.placeHolder.isFirst = false;
        }
        this.moveDraggingGhost(event)
        this.setHoveredTreeChild(event) 
      }
      this.hoveringOnTree(event)      

      // sort
      if(this.isDraggedOverOriginalLocation) {
        const sortableNodes = Array.from(document.querySelectorAll('.node')).filter(node => node.style.display !== "none")
        if(sortableNodes && this.dragging && this.hoveredTreeChild.element) {
          const belowNode = sortableNodes.reduce((closestNode, sortableNode) => {
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
          this.belowNode.element = belowNode

          const aboveNode = sortableNodes.reduce((closestNode, sortableNode) => {
            const nodeLocation = sortableNode.getBoundingClientRect()
            const nodeLabelLocation = sortableNode.querySelector('a').getBoundingClientRect()
            const offsetY = event.pageY - (nodeLocation.top + nodeLabelLocation.height / 2)
            if(offsetY >= 0 && offsetY < closestNode.offsetY) {
              return {
                offsetY: offsetY,
                element: sortableNode
              }
            } else {
              return closestNode
            }
          }, { offsetY: Number.POSITIVE_INFINITY }).element
          this.aboveNode.element = aboveNode

          this.placeHolder.element.remove()
          if (belowNode == undefined || this.aboveNode.element && this.belowNode.element && this.aboveNode.element.dataset.hierarchy > this.belowNode.element.dataset.hierarchy && this.belowNode.element.parentNode != this.hoveredTreeChild.element) {
            this.setPlaceHolder(this.hoveredTreeChild.element.dataset.hierarchy)
            this.hoveredTreeChild.element.appendChild(this.placeHolder.element)
          } else {
            this.setPlaceHolder(belowNode.dataset.hierarchy)
            belowNode.parentNode.insertBefore(this.placeHolder.element, belowNode)
          }        
          if(this.isInsideElement(this.placeHolder.element, event)){
            this.placeHolder.element.classList.add("hoveredPlaceHolder")
          }
        }
      }
    },
    setPlaceHolder(hierarchy) {
      this.placeHolder.element = document.createElement("li")
      this.placeHolder.element.classList.add("place-holder")
      this.placeHolder.element.appendChild(document.createTextNode("▶︎"))
      this.placeHolder.element.style.textIndent = (hierarchy * 16) + "px"   
    },
    createDraggingGhost() {
      this.draggingGhost.element = document.createElement("li")
      this.draggingGhost.element.classList.add("draggingGhost")
      this.draggingGhost.element.appendChild(document.createTextNode(this.dragged.element.children[0].textContent))
      this.draggingGhost.element.style.position = "absolute"
      this.$refs.draggingGhost.appendChild(this.draggingGhost.element)
      this.draggingGhost.element.style.top = `${this.dragged.top}px`
      this.draggingGhost.element.style.left = `${this.dragged.left}px`
    },
    moveDraggingGhost(event) {
      const moveX = event.pageX - this.dragged.pageX
      const moveY = event.pageY - this.dragged.pageY
      this.draggingGhost.element.style.top = `${this.dragged.top + moveY}px`
      this.draggingGhost.element.style.left = `${this.dragged.left + moveX}px`
    },
    hoveringOnTree(event){
      const nodeLabels = Array.from(document.querySelectorAll('.nodeLabel'))
      const hoveredNodeLabel = nodeLabels.find((nodeLabel) => {
        const nodeLabelLocation = nodeLabel.getBoundingClientRect()
        return nodeLabelLocation.top < event.pageY && event.pageY < nodeLabelLocation.bottom
      })
      if(this.hovered.element) {
        this.hovered.element.classList.remove("hovered", "hoveredWithDrag")
      }
      if(hoveredNodeLabel && this.isInsindeTree(event)){
        if(this.dragging) {
          hoveredNodeLabel.classList.add("hoveredWithDrag")
          this.isDraggedOverOriginalLocation = true
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
    isInsideElement(element, event) {
      const elementLocation = element.getBoundingClientRect()
      return (elementLocation.top <= event.pageY && event.pageY <= elementLocation.bottom) && (elementLocation.left <= event.pageX && event.pageX <= elementLocation.right)
    },
    setHoveredTreeChild(event) {
      const treeChildren = Array.from(document.querySelectorAll('.treeChild')).filter(treeChild => treeChild.getBoundingClientRect().height > 0)
      const hoveredTreeChild = treeChildren.reduce((shallowestTreeChild, treeChild) => {
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
      }, { depth: Number.POSITIVE_INFINITY, element: treeChildren[0] }).element
      this.hoveredTreeChild.element = hoveredTreeChild
    },
    mouseUp() {
      this.placeHolder.element.remove()
      this.draggingGhost.element.remove()
      this.dragged.element.style.display = "block"
      this.dragging = false
      this.isDraggedOverOriginalLocation = false
      this.hoveredTreeChild.element = null
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
