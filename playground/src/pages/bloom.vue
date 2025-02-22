<script setup lang="ts">
import { Color, BasicShadowMap, NoToneMapping } from 'three'
import { TresCanvas } from '@tresjs/core'
import { OrbitControls, useTweakPane } from '@tresjs/cientos'
import { BlendFunction, KernelSize } from 'postprocessing'
import { EffectComposer, Bloom } from '@tresjs/post-processing'
import { onMounted, reactive, ref, watch } from 'vue'
import { TresLeches, useControls } from '@tresjs/leches'
import '@tresjs/leches/styles'

const gl = {
  clearColor: '#121212',
  shadows: true,
  alpha: false,
  shadowMapType: BasicShadowMap,
  toneMapping: NoToneMapping,
}

useControls('fpsgraph')

const materialRef = ref()
const {
  intensity,
  blendFunction, 
  resolution,
  kernelSize,
  mipmapBlur,
} = useControls({
  intensity: {
    value: 4.0,
    min: 0,
    max: 10,
    step: 0.1,
  },
  blendFunction: {
    options: Object.keys(BlendFunction).map(key => ({
      text: key,
      value: BlendFunction[key],
    })),
    value: BlendFunction.ADD,
  },
  resolution: {
    value: 256,
    min: 64,
    max: 512,
    step: 64,
  },
  kernelSize: {
    options: Object.keys(KernelSize).map(key => ({
      text: key,
      value: KernelSize[key],
    })),
    value: KernelSize.VERY_SMALL,
  },
  mipmapBlur: true,
})

const { luminanceSmoothing, luminanceThreshold } = useControls('luminance', {
  threshold: {
    value: 0.2,
    min: 0,
    max: 1,
    step: 0.01,
  },
  smoothing: {
    value: 0.3,
    min: 0,
    max: 1,
    step: 0.01,
  },
})

onMounted(() => {
  if (materialRef.value) {
    const { value: emissiveIntensity } = useControls({
      emissiveIntensity: {
        value: 1,
        min: 0,
        max: 10,
        step: 0.1,
      },
    })
    
    watch(emissiveIntensity, (newValue) => {
      materialRef.value.emissiveIntensity = newValue
    })
  }
})
</script>

<template>
  <TresLeches />
  <TresCanvas
    v-bind="gl"
    :disable-render="true"
  >
    <TresPerspectiveCamera
      :position="[5, 5, 5]"
      :look-at="[0, 0, 0]"
    />
    <OrbitControls />
    <TresMesh>
      <TresSphereGeometry :args="[2, 32, 32]" />
      <TresMeshStandardMaterial
        color="hotpink"
        :emissive="new Color('hotpink')"
        :emissive-intensity="9"
      />
    </TresMesh>
    <TresMesh :position="[2, 2, -2]">
      <TresSphereGeometry :args="[2, 32, 32]" />
      <TresMeshStandardMaterial color="hotpink" />
    </TresMesh>
    <TresMesh>
      <TresSphereGeometry :args="[2, 32, 32]" />
      <TresMeshStandardMaterial
        ref="materialRef"
        color="hotpink"
        :emissive="new Color('hotpink')"
        :emissive-intensity="1"
      />
    </TresMesh>
    <TresGridHelper />
    <TresAmbientLight :intensity="0.5" />
    <TresDirectionalLight
      :position="[3, 3, 3]"
      :intensity="2"
    />
    <Suspense>
      <EffectComposer :depth-buffer="true">
        <Bloom 
          :luminance-threshold="luminanceThreshold"
          :luminance-smoothing="luminanceSmoothing"
          :intensity="intensity.value"
          :blend-function="blendFunction.value"
          :kernel-size="kernelSize.value"
          :resolution="resolution.value"
          :mipmap-blur="mipmapBlur.value" 
        />
      </EffectComposer>
    </Suspense>
  </TresCanvas>
</template>
