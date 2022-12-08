<template>
  <div v-on:click="f">
    <div>
      <span>{{title}}</span>
    </div>
    <div>
      <canvas id="canvas" width="900" height="700" style="border: 1px solid black;"></canvas>
    </div>
    <div>
      <button @click="renderOne('Added 5 circles to the canvas one-by-one')">Single objects</button>
      <button @click="renderTwo('Added the 5 circles and a rectangle to a group, then added the group to canvas')">Group</button>
      <button @click="renderThree('Added a group (2 circles), group (3 circles) and a rectangle to canvas')">Single and group</button>
    </div>
    <div>
      <button @click="printInfo">Print info</button>
      <button @click="selectAll">Select all</button>
      <button @click="deselectAll">Deselect all</button>
    </div>
</div>
</template>

<script setup>
import {markRaw,onMounted, onUnmounted, ref} from 'vue'
import {fabric} from 'fabric'

let canvasDomNode = null
let canvas = null
let title = ref('initialized')
const setTitle = s => title.value = s

onMounted(() => {
  canvasDomNode = document.getElementById('canvas')
  canvas = markRaw(new fabric.Canvas(canvasDomNode))
})

const clear = () => { canvas.clear().requestRenderAll() }

const renderOne = (title) => {
  clear(); setTitle(title)
  canvas.add(c1); canvas.add(c2); canvas.add(c3); canvas.add(c4); canvas.add(c5)
  printInfo()
}

const renderTwo = (title) => {
  clear(); setTitle(title)
  let group = new fabric.Group([c1,c2,c3,c4,c5],{ /* options */ })
  group.add(r1)
  canvas.add(group)
  printInfo()
}

const renderThree = (title) => {
  clear(); setTitle(title)
  const g1 = new fabric.Group([c1,c2])
  const g2 = new fabric.Group([c3,c4,c5])
  canvas.add(g1)
  canvas.add(g2)
  canvas.add(r1)
  printInfo()
}

///////////////////////////////////////////////////////////////
//////////////////////////// TESTS ////////////////////////////
///////////////////////////////////////////////////////////////
const selectAll = () => {
  // multi-select
  const selection = new fabric.ActiveSelection(canvas.getObjects(),{canvas})
  canvas.setActiveObject(selection)
  canvas.requestRenderAll()
}

const deselectAll = () => {
  canvas.discardActiveObject()
  canvas.requestRenderAll()
}

const log = (descr,...args) => {
  console.log('-'.repeat(descr.length))
  console.log(descr)
  console.log('-'.repeat(descr.length))
  console.log(...args)
}

const printInfo = () => {
  console.clear()
  console.log(title)

  // this prints the bounding box coords in world space
  log('printing aCoords -> world space')
  canvas.getObjects().forEach(o => {
    console.log(o.aCoords)
  })

  // test various stuff
  log('canvas.getObjects()',canvas.getObjects())
  log('canvas.getActiveObject()',canvas.getActiveObject())
  log('canvas.getActiveObjects()',canvas.getActiveObjects())
}

onUnmounted(() => {
  //
})

const c1 = new fabric.Circle({
  radius: 100,
  fill: 'red',
  originX: 'center',
  originY: 'center',
  left: 100,
  top: 100
})

const c2 = new fabric.Circle({
  radius: 100,
  fill: 'green',
  originX: 'center',
  originY: 'center',
  left: 700,
  top: 100
})

const c3 = new fabric.Circle({
  radius: 100,
  fill: 'blue',
  originX: 'center',
  originY: 'center',
  left: 700,
  top: 500
})

const c4 = new fabric.Circle({
  radius: 100,
  fill: 'magenta',
  originX: 'center',
  originY: 'center',
  left: 100,
  top: 500
})

const c5 = new fabric.Circle({
  radius: 100,
  fill: 'cyan',
  originX: 'center',
  originY: 'center',
  left: 400,
  top: 300
})

const r1 = new fabric.Rect({
  width: 100,
  height: 100,
  fill: 'green',
  // originX: 'center',
  // originY: 'center'
})
</script>
