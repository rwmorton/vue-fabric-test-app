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
      <button @click="renderOne('Added 5 circles to the canvas one-by-one')">Single objects</button>
      <button @click="renderTwo('Added the 5 circles and a rectangle to a group, then added the group to canvas')">Group</button>
      <button @click="renderThree('Added a group (2 circles), group (3 circles) and a rectangle to canvas')">Single and group</button>
      <button @click="renderFour('TESTING TRANSFORMS')">Test transforms</button>
      <button @click="renderFive('TESTING REAL GROUP')">Test real group</button>
    </div>
    <div>
      <span>ACTIONS </span>
      <button @click="printInfo">Print info</button>
      <button @click="selectAll">Select all</button>
      <button @click="deselectAll">Deselect all</button>
      <button @click="groupSelection">Group selection</button>
      <button @click="ungroupSelection">Ungroup selection</button>
      <button @click="test">TEST</button>
      <button @click="clearConsole">CLEAR CONSOLE</button>
      <button @click="sanityCheck">SANITY CHECK</button>
      <button @click="setSelectedToIdentity">Set selected to identity</button>
    </div>
    <div>
      <span>SAFE ACTIONS </span>
      <button @click="safeGroupSelection">Group selection</button>
      <button @click="ungroupSelection">Ungroup selection</button>
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

  renderThree() // default scene

  canvas.on('selection:updated',() => {
    //
    console.log('selection:updated')
    const activeObject = canvas.getActiveObject()
    console.log(`activeObject.type === ${activeObject?.type}`)
  })
})

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
    fabric.util.applyTransformToObject(object,[1,0,0,1,0,0])
  })
}

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
  const g1 = new fabric.Group([c1,c2,c4])
  g1.set({
    name: 'Group #1 (3 circles)'
  })
  const g2 = new fabric.Group([c3,c5])
  g2.set({
    name: 'Group #2: (2 circles)'
  })
  canvas.add(g1)
  canvas.add(g2)
  canvas.add(r1)
  printInfo()
}

const renderFour = (title) => {
  clear(); setTitle(title);
  const circle = new fabric.Circle({
    originX: 'center',
    originY: 'center',
    radius: 100,
    left: 450,
    top: 350,
    // opacity: 0,
    fill: 'rgba(0,0,0,0)',
    stroke: 'red',
    strokeWidth: 5,
    name: 'Circle - CHILD'
  })
  const rectangle = new fabric.Rect({
    originX: 'center',
    originY: 'center',
    width: 600,
    height: 400,
    left: 450,
    top: 350,
    // opacity: 0,
    fill: 'rgba(0,0,0,0)',
    stroke: 'green',
    strokeWidth: 5,
    name: 'Rectangle - PARENT'
  })
  canvas.add(rectangle)
  canvas.add(circle)

  console.clear()
  printObjectInfo(circle)
}

const renderFive = (title) => {
  clear(); setTitle(title);
  const circle = new fabric.Circle({
    originX: 'center',
    originY: 'center',
    radius: 100,
    left: 450,
    top: 350,
    // opacity: 0,
    fill: 'rgba(0,0,0,0)',
    stroke: 'red',
    strokeWidth: 5,
    name: 'Circle'
  })
  const rectangle = new fabric.Rect({
    originX: 'center',
    originY: 'center',
    width: 600,
    height: 400,
    left: 450,
    top: 350,
    // opacity: 0,
    fill: 'rgba(0,0,0,0)',
    stroke: 'green',
    strokeWidth: 5,
    name: 'Rectangle'
  })
  const group = new fabric.Group([rectangle,circle])
  group.set({name: 'GROUP'})
  canvas.add(group)

  console.clear()
  printObjectInfo(group)
  printObjectInfo(circle)
  printObjectInfo(rectangle)
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

const groupSelection = () => {
  const activeObject = canvas.getActiveObject()
  if(!activeObject) {
    console.log('no activeObject - returning')
    return
  } else if(activeObject.type !== 'activeSelection') {
    console.log("activeObject.type !== 'activeSelection' - returning")
    console.log(`activeObject.type === ${activeObject.type}`)
    return
  } else { // type === 'activeSelection'
    // activeObject.toGroup() // latest API - doens't work in current patch version

    // get all selected objects
    const objs = canvas.getActiveObjects() // same as canvas._activeObject._objects

    console.log(`there are ${objs.length} selected objects!`)

    // get coords of activeObject - use to update the group coords
    const activeObject = canvas.getActiveObject()
    const coords = activeObject.oCoords
    const xformMatrix = activeObject.calcTransformMatrix()

    console.log('transform matrix')
    console.log(xformMatrix)
    //

    canvas.remove(...objs) // Remove the objects from the canvas itself, since we're going to re-add them as part of a group    
    //
    // !!! NB: must make sure it is passed as new fabric.Object(o) !!!
    //
    const group = new fabric.Group(objs.map(o => {
      const obj = new fabric.Object(o)

      // is object a group?
      if(o.type === 'group') {
        console.log('got a group in the selection - apply xformMatrix to it')
        // fabric.util.applyTransformToObject(obj,o.calcTransformMatrix())
        // fabric.util.applyTransformToObject(obj,o.calcOwnMatrix())
        // obj.set({
        //   left: o.left,
        //   top: o.top
        // })
      }

      return obj
    }))
    group.set({name: 'GROUP FROM SELECTION'})

    // update coords to maintain selection
    group.set({
      left: coords.tl.x,
      top: coords.tl.y
    })

    canvas.add(group)
    // canvas.setActiveObject(group)

    // canvas.requestRenderAll()
    // canvas.renderAll()
  }
}

const safeGroupSelection = () => {
  const activeObject = canvas.getActiveObject()

  if(!activeObject || activeObject?.type !== 'activeSelection') {
    return
  } else { // type === 'activeSelection'
    // activeObject.toGroup() // latest API - doens't work in current patch version

    // get all selected objects
    const objects = canvas.getActiveObjects() // same as canvas._activeObject._objects

    //
    // CURRENT LIMIATION: cannot group another group (YET!)
    //
    for(let i=0; i<objects.length; i++) {
      if(objs[i].type === 'group') return
    }

    canvas.remove(...objects) // Remove the objects from the canvas itself, since we're going to re-add them as part of a group    
    //
    // !!! NB: must make sure it is passed as new fabric.Object(o) !!!
    //
    const group = new fabric.Group(objects.map(object => new fabric.Object(object)))

    // update group coords to that of activeObject
    const coords = activeObject.oCoords
    group.set({
      left: coords.tl.x,
      top: coords.tl.y
    })

    canvas.add(group)
  }
}

const ungroupSelection = () => {

  log('UNGROUPING!')

  const group = canvas.getActiveObject()

  // debug
  printObjectInfo(group)
  if(group && group.type === 'group') {
    group.getObjects().forEach(o => {
      printObjectInfo(o)
      const gM = group.calcOwnMatrix()
      const oM = o.calcOwnMatrix()
      console.log(`groupM * objectM = ${fabric.util.multiplyTransformMatrices(gM,oM)}`)
    })
  }
  //

  /*
  For each object in the group we need to make a copy of it.
  We then multiply that objects local matrix by the group local (world) matrix
  */
  if(group && group.type === 'group') {
    const objects = group.getObjects()

    console.log(`there are ${objects.length} objects in the group...`)
    console.log(objects)

    const groupMatrix = group.calcTransformMatrix() // group.calcOwnMatrix() // [1,0,0,1,group.left,group.top]
    const coords = group.aCoords

    console.log(`groupMatrix = ${groupMatrix}`)
    // console.log(`qrDecompose groupMatirx =`,fabric.util.qrDecompose(groupMatrix))

    objects.forEach(object => {
      //
      const objMatrix = object.calcTransformMatrix() // object.calcOwnMatrix()
      //
      console.log(`objMatrix = ${objMatrix}`)
      group.remove(object)

      const identity = [1,0,0,1,0,0]
      const xform = fabric.util.multiplyTransformMatrices(
        groupMatrix,
        objMatrix
      )

      // fabric.util.applyTransformToObject(
      //   object,
      //   xform
      // )

      fabric.util.removeTransformFromObject(object,groupMatrix)
      // fabric.util.applyTransformToObject(object,xform)

      object.set({
        left: (object.left + group.left) + object.width * 0.5,
        top: (object.top + group.top) + object.height * 0.5
      })

      printObjectInfo(object)

      // add object back to the canvas
      canvas.add(object)

      // // DEBUG HELPER!
      // const circle = new fabric.Circle({
      //   radius: 50,
      //   fill: 'BLUE'
      // })
      // fabric.util.applyTransformToObject(circle,groupMatrix)
      // canvas.add(circle)

    })

    // canvas.discardActiveObject()
  } else {
    // NOT A GROUP!
  }
}

const sanityCheck = () => {
  //
  console.log("TODO!")
  const objects = canvas.getObjects()
  console.log(objects)
  const parent = objects[0]
  const child = objects[1]
  printObjectInfo(child)

  // apply inverse of parent to child to group
  const parentXform = parent.calcTransformMatrix()
  const parentInverse = fabric.util.invertTransform(parentXform)
  const childXform = child.calcTransformMatrix()
  const newChildXform = fabric.util.multiplyTransformMatrices(parentInverse,childXform)

  console.log(`parentXform = ${parentXform}`)
  console.log(`parentInverse = ${parentInverse}`)
  console.log(`the child is still in world space, it's transform matrix = ${childXform}`)
  console.log('now making child a REAL child of parent...')
  console.log(`REAL child xform = parentInverse * childXform = ${newChildXform}`)

  console.log(`now ungroup the child - move it to world space`)
  console.log(`now child is back in world space: xform = parentXform * childXform = ${fabric.util.multiplyTransformMatrices(parentXform,newChildXform)}`)
  //

  // make copy of child in world space
  const childCopyWorld = new fabric.Circle({
    color: 'red',
    radius: 100
  })
  fabric.util.applyTransformToObject(
    childCopyWorld,
    fabric.util.multiplyTransformMatrices(parentXform,newChildXform)
  )
  canvas.add(childCopyWorld)
}

const printBoundingBox = () => {
  //
  // const activeSelection = canvas.getActiveObjects()
  const activeSelection = canvas.getActiveObject()
  console.log(activeSelection)
}

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
  console.log('worldMatrix',obj.calcTransformMatrix())
  // console.log('transformMatrix',obj.calcTransformMatrix())
  // console.log('transformMatrix inverse',fabric.util.invertTransform(obj.calcTransformMatrix()))
  console.log('calcOwnMatrix',obj.calcOwnMatrix())
}

const printInfo = () => {
  console.clear()
  console.log(title)

  // this prints the bounding box coords in world space
  log('printing aCoords -> world space')
  canvas.getObjects().forEach(o => {
    console.log(o.aCoords)
    console.log(o.left,o.top,o.width,o.height)
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
  radius: 10,
  fill: 'red',
  originX: 'center',
  originY: 'center',
  left: 380,
  top: 280,
  name: 'Circle #1'
})

const c2 = new fabric.Circle({
  radius: 10,
  fill: 'green',
  originX: 'center',
  originY: 'center',
  left: 420,
  top: 320,
  name: 'Circle #2'
})

const c3 = new fabric.Circle({
  radius: 10,
  fill: 'blue',
  left: 0,
  top: 0,
  originX: 'center',
  originY: 'center',
  left: 510,
  top: 280,
  name: '!!!!!!!!!! Circle #3 !!!!!!!!!!!!!'
})

const c4 = new fabric.Circle({
  radius: 10,
  fill: 'magenta',
  originX: 'center',
  originY: 'center',
  left: 380,
  top: 340,
  name: 'Circle #4'
})

const c5 = new fabric.Circle({
  radius: 10,
  fill: 'cyan',
  originX: 'center',
  originY: 'center',
  left: 510,
  top: 340,
  name: 'Circle #5'
})

const r1 = new fabric.Rect({
  width: 100,
  height: 100,
  fill: 'green',
  // originX: 'center',
  // originY: 'center',
  name: 'Rectangle #1'
})
</script>
