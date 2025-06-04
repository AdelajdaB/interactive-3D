<script setup>
  import { onMounted, ref } from 'vue'
  import * as THREE from 'three'
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
  import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader.js'
  
  const canvasEl = ref()
  let camera, scene, renderer
  
  onMounted(() => {
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

            scene.add(gltf.scene)

            const size = box.getSize(new THREE.Vector3());
            const maxDim = Math.max(size.x, size.y, size.z);
            camera.position.z = maxDim * 0.8; 
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
  })
  
  function zoomTo(index) {
    const positions = [
      { x: 0, y: 0, z: 5 },
      { x: 1, y: 1, z: 4 },
      { x: -1, y: 0, z: 4 },
    ]
    const target = positions[index]
    if (!target) return
  
    camera.position.set(target.x, target.y, target.z)
  }
  defineExpose({ zoomTo })
</script>

<template>
    <canvas ref="canvasEl" class="w-full h-full" />
</template>
    