<template>
  <div ref="globeContainer" class="globe-container"></div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import * as THREE from 'three'
import Globe from 'three-globe'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const globeContainer = ref(null)

let contriesColor = '#A8D8FF';
let borderColor = '#2C2F33';
let pointColor = '#2C2F33';
let oceanColor = '#007BFF';
let contriesColorHover = '#6A5ACD';



onMounted(async () => {
  const scene = new THREE.Scene()

  scene.add(new THREE.AmbientLight(0xffffff, 0.9))
  const dirLight = new THREE.DirectionalLight(0xffffff, 0.6)
  dirLight.position.set(5, 3, 5)
  scene.add(dirLight)

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 250

  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  globeContainer.value.appendChild(renderer.domElement)

  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enablePan = false
  controls.minDistance = 80
  controls.maxDistance = 400

  // Chargement des pays
  const countries = await fetch('/world.geojson').then(r => r.json())

  const globe = new Globe()
      // Océans stylisés
      .globeMaterial(
          new THREE.MeshPhongMaterial({
            color: oceanColor,
            flatShading: true
          })
      )

      // Pays
      .polygonsData(countries.features)
      .polygonAltitude(0.01)
      .polygonCapColor(d => {
        // couleur par pays (modifiable)
        if (d.properties.name === 'France') return contriesColorHover
        return contriesColor
      })
      .polygonSideColor(() => contriesColor)
      .polygonStrokeColor(() => borderColor)

      // Pings
      .pointsData([
        { lat: 48.8566, lng: 2.3522, size: 0.25, color: pointColor }
      ])
      .pointColor(d => d.color)
      .pointRadius(d => d.size)
  scene.add(globe)

  function animate() {
    requestAnimationFrame(animate)
    renderer.render(scene, camera)
    controls.update()
  }
  animate()
})
</script>

<style>
.globe-container {
  width: 100%;
  height: 100vh;
  background: #87ceeb;
}
</style>
