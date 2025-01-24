<script setup lang="ts">
import { ref, shallowRef, toRefs, watch } from 'vue'
import type { BufferGeometry, LineSegments } from 'three'
import { EdgesGeometry } from 'three'
import type { TresColor, TresGroup } from '@tresjs/core'

export interface EdgesProps {
  color?: TresColor
  threshold?: number
}

const props = withDefaults(defineProps<EdgesProps>(), {
  color: '#ff0000',
  threshold: 15,
})

const { color, threshold } = toRefs(props)

const mainGroupRef = shallowRef<TresGroup>(null)
const meshes = ref<THREE.Mesh[]>([])

defineExpose({
  instance: mainGroupRef,
})

watch(mainGroupRef, (group) => {
  if (!mainGroupRef.value) { return }

  console.log('mainGroupRef', mainGroupRef.value.parent)

  mainGroupRef.value.traverse((child) => {
    if (child.isMesh) {
      meshes.value.push(child as THREE.Mesh)
    }
  })
})

watch(
  () => [meshes.value, threshold.value],
  () => {
    meshes.value.forEach((mesh) => {
      const parent = mesh.parent

      if (parent) {
        const geometry = mesh.geometry

        const lineGeom = new EdgesGeometry(geometry, threshold.value)
        const line = new LineSegments(lineGeom, new THREE.LineBasicMaterial({ color: props.color }))
        line.position.copy(mesh.position)
        line.scale.copy(mesh.scale)
        line.rotation.copy(mesh.rotation)

        const thickLineGeom = new LineSegmentsGeometry().fromEdgesGeometry(lineGeom)
        const thickLines = new LineSegments2(thickLineGeom, new LineMaterial({ color: props.color, linewidth: 3 }))
        thickLines.position.copy(mesh.position)
        thickLines.scale.copy(mesh.scale)
        thickLines.rotation.copy(mesh.rotation)

        parent.remove(mesh)
        parent.add(line)
        parent.add(thickLines)
      }
    })
  },
)
</script>

<template>
  <TresGroup
    ref="mainGroupRef"
    v-bind="$attrs"
  >
    <template v-for="mesh in meshes" :key="mesh.uuid">
      <TresEdgesGeometry :geometry="mesh.geometry" :threshold="props.threshold">
        <TresLineSegments :geometry="mesh.geometry" :material="new THREE.LineBasicMaterial({ color: props.color })" />
      </TresEdgesGeometry>
    </template>
  </TresGroup>
</template>
