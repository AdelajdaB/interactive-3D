<script setup>
  import { onMounted, ref } from 'vue'
  import * as THREE from 'three'
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
  import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader.js'
  
  const canvasEl = ref()
  let camera, scene, renderer
  let model 

  // Define camera positions for each point
  const cameraPositions = [
    { position: new THREE.Vector3(0, 0, 100), lookAt: new THREE.Vector3(0, 0, 0) }, // Initial view
    { position: new THREE.Vector3(-23, 5, 10), lookAt: new THREE.Vector3(-23, 0, 0) }, // Transition with zoom out
    { position: new THREE.Vector3(-46, 0, 10), lookAt: new THREE.Vector3(-46, 0, 4) }, // Fore Mast
    { position: new THREE.Vector3(-23, 5, 40), lookAt: new THREE.Vector3(-23, 0, 0) }, // Transition with zoom out
    { position: new THREE.Vector3(0, -25, 30), lookAt: new THREE.Vector3(0, -18, 0) }, // Keel (bottom center)
    { position: new THREE.Vector3(0, -15, 60), lookAt: new THREE.Vector3(0, -2, 0) }, // Transition with zoom out
    { position: new THREE.Vector3(30, 10, 10), lookAt: new THREE.Vector3(30, 10, 4) }, // Bridge
    { position: new THREE.Vector3(15, 5, 30), lookAt: new THREE.Vector3(0, 5, 0) }, // Transition with zoom out
    { position: new THREE.Vector3(43, -15, 20), lookAt: new THREE.Vector3(43, -10, 4) }, // Rudder
    { position: new THREE.Vector3(15, 5, 80), lookAt: new THREE.Vector3(0, 0, 0) }, // Transition back to initial
    { position: new THREE.Vector3(3, 1, 80), lookAt: new THREE.Vector3(0, 0, 0) }  // Back to initial view
]

  const debugMode = ref(true)
const cameraDebug = ref({
  position: { x: 0, y: 0, z: 0 },
  lookAt: { x: 0, y: 0, z: 0 }
})

  // Handle scroll animation
  const handleScroll = () => {
    if (!camera || !scene) return
    
    const scrollPosition = window.scrollY
    const windowHeight = window.innerHeight
    const documentHeight = document.documentElement.scrollHeight
    const scrollProgress = (scrollPosition / (documentHeight - windowHeight))
    
    // Calculate which segment we're in (0-3)
    const totalSegments = cameraPositions.length - 1
    const segment = Math.min(Math.floor(scrollProgress * totalSegments), totalSegments - 1)
    const segmentProgress = (scrollProgress * totalSegments) % 1
    
    // Interpolate between current and next position
    const currentPos = cameraPositions[segment].position
    const nextPos = cameraPositions[segment + 1].position
    const currentLookAt = cameraPositions[segment].lookAt
    const nextLookAt = cameraPositions[segment + 1].lookAt
    
    // Lerp camera position
    camera.position.lerpVectors(currentPos, nextPos, segmentProgress)
    
    // Lerp lookAt point
    const currentTarget = new THREE.Vector3()
    currentTarget.lerpVectors(currentLookAt, nextLookAt, segmentProgress)
    camera.lookAt(currentTarget)

    if (debugMode.value) {
    cameraDebug.value = {
      position: {
        x: Number(camera.position.x.toFixed(2)),
        y: Number(camera.position.y.toFixed(2)),
        z: Number(camera.position.z.toFixed(2))
      },
      lookAt: {
        x: Number(currentTarget.x.toFixed(2)),
        y: Number(currentTarget.y.toFixed(2)),
        z: Number(currentTarget.z.toFixed(2))
      }
    }
  }
  }

  onMounted(() => {
    window.scrollTo(0, 0)

    scene = new THREE.Scene()
    camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
    renderer = new THREE.WebGLRenderer({ 
        canvas: canvasEl.value, 
        alpha: true,
        analysis: true,
        precision: 'highp', 
    })

    renderer.shadowMap.enabled = true
    renderer.shadowMap.type = THREE.PCFSoftShadowMap
    renderer.setPixelRatio(window.devicePixelRatio)
    renderer.toneMapping = THREE.ACESFilmicToneMapping
    renderer.toneMappingExposure = 1.0
    renderer.outputEncoding = THREE.sRGBEncoding

    new RGBELoader().load(
        'models/scene.hdr', 
        function(texture) {
            texture.mapping = THREE.EquirectangularReflectionMapping
            scene.environment = texture

            loadModel()
        }
    )

    window.addEventListener('scroll', handleScroll)

    function loadModel(){
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
      directionalLight.position.set(139.78, 31.8, 1.93)
      directionalLight.followCamera = true
      scene.add(directionalLight)

      renderer.setSize(window.innerWidth * 0.8, window.innerHeight)
      camera.position.z = 5
    
      const loader = new GLTFLoader()
      loader.load(
          'models/cargo_ship.glb', 
          (gltf) => {
              gltf.scene.traverse((node) => {
                  if (node.isMesh) {
                      node.castShadow = true;
                      node.receiveShadow = true;

                      if(node.material) {
                          node.material.needsUpdate = true;
                          node.material.metalness = 1;
                          node.material.roughness = 1;
                      }
                  }
              })

              const box = new THREE.Box3().setFromObject(gltf.scene);
              const center = box.getCenter(new THREE.Vector3());
              gltf.scene.position.sub(center);

              gltf.scene.scale.x = -1;

              model = gltf.scene
              scene.add(model)

              const size = box.getSize(new THREE.Vector3());
              const maxDim = Math.max(size.x, size.y, size.z);
              camera.position.z = maxDim * 0.8; 

              // Initial camera position
              camera.position.copy(cameraPositions[0].position)
              camera.lookAt(cameraPositions[0].lookAt)
          },
          (progress) => {
              // console.log('Loading Progress:', Math.round((progress.loaded / progress.total) * 100) + '%')
          },
          (error) => {
              console.error('Error loading model:', error)
          }
      )
    
      animate()   
    }
  
    function animate() {
      requestAnimationFrame(animate)
      renderer.render(scene, camera)
    }

    return () => {
    window.removeEventListener('scroll', handleScroll)
    window.removeEventListener('load', () => {
      window.scrollTo(0, 0)
    })
  }
  })

  window.addEventListener('load', () => {
    window.scrollTo(0, 0)
    if (camera && scene) {
      camera.position.copy(cameraPositions[0].position)
      camera.lookAt(cameraPositions[0].lookAt)
    }
  })
  
  function zoomTo(index) {
    const target = cameraPositions[index]
    if (!target) return
    
    // camera.position.copy(target.position)
    // camera.lookAt(target.lookAt)
  }

  defineExpose({ zoomTo })
</script>

<template>
    <canvas ref="canvasEl" class="w-full h-full mr-16" />

        <!-- Debug Overlay -->
    <!-- <div v-if="debugMode" class="fixed top-0 left-0 bg-black bg-opacity-75 text-white p-4 font-mono text-sm">
        <div class="mb-2">
            <div>Camera Position:</div>
            <div>X: {{ cameraDebug.position.x }}</div>
            <div>Y: {{ cameraDebug.position.y }}</div>
            <div>Z: {{ cameraDebug.position.z }}</div>
        </div>
        <div>
            <div>Look At:</div>
            <div>X: {{ cameraDebug.lookAt.x }}</div>
            <div>Y: {{ cameraDebug.lookAt.y }}</div>
            <div>Z: {{ cameraDebug.lookAt.z }}</div>
        </div>
    </div> -->
</template>

<style scoped>

.debug-overlay {
  z-index: 1000;
}
</style>
