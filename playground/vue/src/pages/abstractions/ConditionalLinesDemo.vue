<script setup lang="ts">
import { TresCanvas } from '@tresjs/core'
import { ConditionalLines, Octahedron, OrbitControls, useGLTF } from '@tresjs/cientos'
import { TresLeches, useControls } from '@tresjs/leches'
import '@tresjs/leches/styles'

const gl = {
  powerPreference: 'high-performance' as WebGLPowerPreference,
  precision: 'highp',
  clearColor: '#F6B03B',
}

const coucouTest = ref(null)

const model = await useGLTF('https://raw.githubusercontent.com/Tresjs/assets/main/models/gltf/positional-audio/ping-pong.glb', { draco: true })

const { enabled, edgeColor, edgeThreshold } = useControls({
  enabled: { value: true, type: 'boolean', label: 'Enabled' },
  edgeColor: { value: '#292929', type: 'color', label: 'Color' },
  edgeThreshold: {
    label: 'Threshold Angle',
    value: 15,
    min: 1,
    max: 100,
    step: 1,
  },
})

watch(coucouTest, (value) => {
  console.log('coucouTest', value)
})
</script>

<template>
  <TresLeches />

  <TresCanvas
    v-bind="gl"
  >
    <TresPerspectiveCamera
      :position="[0, 2, 5]"
    />

    <OrbitControls
      make-default
    />

    <TresGridHelper
      :args="[10, 10]"
      :position-y="-.5"
    />

    <Suspense>
      <primitive
        ref="coucouTest"
        :scale="[.2, .2, .2]"
        :position="[0, -1.15, 0]"
        receive-shadow
        :object="model.scene"
      >
        <TresMeshBasicMaterial :color="0xFF0000" />
      </primitive>
    </Suspense>

    <TresAmbientLight />
    <TresDirectionalLight />

    <Octahedron :args="[1, 0]" :position="[0, 0, 0]">
      <TresMeshBasicMaterial color="#f6f6f6" />

      <ConditionalLines
        v-if="enabled.value"
        :scale="1.1"
        :threshold="edgeThreshold.value"
      >
        <TresMeshBasicMaterial
          :color="edgeColor.value"
        />
      </ConditionalLines>
    </Octahedron>
  </TresCanvas>
</template>
