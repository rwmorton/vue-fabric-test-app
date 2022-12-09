<template>
  <div v-on:click="f">
    <div>
      <span>{{title}}</span>
    </div>
    <div>
      <canvas id="canvas" width="900" height="700" style="border: 1px solid black;"></canvas>
    </div>
    <div>
      <span>SCENES </span>
      <button @click="toggleCreateMode">{{createMode ? 'Exit create mode' : 'Enter create mode'}}</button>
    </div>
    <div>
      <span>ACTIONS </span>
      <button @click="selectAll">Select all</button>
      <button @click="groupSelection">Group selection</button>
      <button @click="ungroupSelection">Ungroup selection</button>
      <button @click="test">TEST</button>
      <button @click="clearConsole">CLEAR CONSOLE</button>
      <button @click="sanityCheck">SANITY CHECK</button>
    </div>
</div>
</template>

<script setup>
import {markRaw,onMounted, onUnmounted, ref} from 'vue'
import {fabric} from 'fabric'

let canvasDomNode = null
let canvas = null
let title = ref('initialized')
let createMode = ref(false)
let scene = []
const setTitle = s => title.value = s

const toggleCreateMode = () => {
  createMode.value = !createMode.value
}

onMounted(() => {
  canvasDomNode = document.getElementById('canvas')
  canvas = markRaw(new fabric.Canvas(canvasDomNode))

  canvas.on('selection:updated',() => {
    //
    console.log('selection:updated')
    const activeObject = canvas.getActiveObject()
    console.log(`activeObject.type === ${activeObject?.type}`)
  })

  canvas.on('mouse:down',(e) => {    
    const x = e.pointer.x
    const y = e.pointer.y

    if(createMode.value) {
      const circle = new fabric.Circle({
        left: x,
        top: y,
        fill: 'red',
        radius: 10,
        name: `Shape #${scene.length}`
      })
      scene.push(circle)
      canvas.add(circle)

      console.log(`added ${circle.name} at (${x},${y})`)
    }
  })
})

const groupSelection = () => {
  const activeObject = canvas.getActiveObject()

  if(!activeObject || activeObject?.type !== 'activeSelection') {
    return
  } else { // type === 'activeSelection'
    const objects = canvas.getActiveObjects()

    canvas.remove(...objects)

    const group = new fabric.Group()

    objects.forEach(object => {
      group.addWithUpdate(object)
    })
  
    group.set({
      left: activeObject.left,
      top: activeObject.top,
    })

    canvas.add(group)
  }
}

const ungroupSelection = () => {
  const group = canvas.getActiveObject()
  
  if(group && group.type === 'group') {
    const objects = group.getObjects()

    objects.forEach(object => {
      group.removeWithUpdate(object)
      canvas.add(object)
    })

    canvas.discardActiveObject()
  } else {
    // NOT A GROUP!
    // show notification?
  }

  canvas.requestRenderAll()
}

const selectAll = () => {
  canvas.discardActiveObject()
  canvas.setActiveObject(new fabric.ActiveSelection(canvas.getObjects(),{canvas}))
  canvas.requestRenderAll()
}

const removeWorldTransform = () => {
  const selection = canvas.getActiveObject()
  console.log(selection)

  const xform = selection.calcTransformMatrix()
  const xformInverse = fabric.util.invertTransform(xform)

  console.log(xform)
  console.log(xformInverse)

  const mult = fabric.util.multiplyTransformMatrices(xform,xformInverse)

  fabric.util.applyTransformToObject(selection,mult)

  console.log('after',selection.calcTransformMatrix())

  canvas.requestRenderAll()
}

const clearConsole = () => { console.clear() }

const test = () => {
  // console.clear()
  // r1.set({
  //   opacity: 0
  // })
  // canvas.requestRenderAll()
  const objs = canvas.getActiveObjects()
  objs.forEach(o => {
    if(o.type === 'group') {
      console.log('object is a group - printing its info')
      printObjectInfo(o)
      console.log('now print info for each child of the group (not recursive!)')
      o.getObjects().forEach(groupO => {
        printObjectInfo(groupO)
      })
    } else {
      printObjectInfo(o)
    }
  })
}

const setSelectedToIdentity = () => {
  canvas.getActiveObjects().forEach(object => {
    // fabric.util.applyTransformToObject(object,[1,0,0,1,0,0])
    object.set({
      angle: 0,
      scaleX: 1,
      scaleY: 1,
      skewX: 0,
      skewY: 0,
      translateX: 0,
      translateY: 0
    })
    console.log(object)
    console.log(object.transformMatrix)
  })
}

const clear = () => { canvas.clear().requestRenderAll() }

const log = (descr,...args) => {
  console.log('-'.repeat(descr.length))
  console.log(descr)
  console.log('-'.repeat(descr.length))
  console.log(...args)
}

const printObjectInfo = obj => {
  if(obj.name) {
    console.log('-'.repeat(obj.name.length))
    console.log(obj.name)
    console.log('-'.repeat(obj.name.length))
  }
  console.log('aCoords',obj.aCoords)
  console.log('[left,top] = [',obj.left,',',obj.top,']')
  console.log('[width,height] = [',obj.width,',',obj.height,']')
  const worldMatrix = obj.calcTransformMatrix()
  console.log('worldMatrix',worldMatrix)
  // console.log('transformMatrix',obj.calcTransformMatrix())
  // console.log('transformMatrix inverse',fabric.util.invertTransform(obj.calcTransformMatrix()))
  console.log('calcOwnMatrix',obj.calcOwnMatrix())
  console.log(`qrDecompose(worldMatrix) = `,fabric.util.qrDecompose(worldMatrix))
  console.log(`obj.group = `,obj.group)
  console.log(`obj.__owningGroup = `,obj.__owningGroup)
}

onUnmounted(() => {
  //
})
</script>
