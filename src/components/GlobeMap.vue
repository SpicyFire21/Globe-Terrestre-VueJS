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
let pointSize = 0.25;
let hoveredCountry = null


// les points a ajouter sur le globe, certains paramètres sont a ajouter pour modifier l'apparence
let points = [
  { lat: 48.8566, lng: 2.3522, size: pointSize, color: pointColor },
  { lat: 40.7128, lng: -74.0060, size: pointSize, color: pointColor },
  { lat: 35.6895, lng: 139.6917, size: pointSize, color: pointColor },
  { lat: 51.5074, lng: -0.1278, size: pointSize, color: pointColor },
  { lat: 52.5200, lng: 13.4050, size: pointSize, color: pointColor },
  { lat: 41.9028, lng: 12.4964, size: pointSize, color: pointColor },
  { lat: 55.7558, lng: 37.6173, size: pointSize, color: pointColor },
  { lat: 34.0522, lng: -118.2437, size: pointSize, color: pointColor },
  { lat: 37.7749, lng: -122.4194, size: pointSize, color: pointColor },
  { lat: 25.2048, lng: 55.2708, size: pointSize, color: pointColor },

  { lat: -33.8688, lng: 151.2093, size: pointSize, color: pointColor },
  { lat: -37.8136, lng: 144.9631, size: pointSize, color: pointColor },
  { lat: -23.5505, lng: -46.6333, size: pointSize, color: pointColor },
  { lat: -34.6037, lng: -58.3816, size: pointSize, color: pointColor },
  { lat: 19.4326, lng: -99.1332, size: pointSize, color: pointColor },
  { lat: 4.7110, lng: -74.0721, size: pointSize, color: pointColor },
  { lat: -12.0464, lng: -77.0428, size: pointSize, color: pointColor },
  { lat: 1.3521, lng: 103.8198, size: pointSize, color: pointColor },
  { lat: 13.7563, lng: 100.5018, size: pointSize, color: pointColor },
  { lat: 22.3193, lng: 114.1694, size: pointSize, color: pointColor },

  { lat: 28.6139, lng: 77.2090, size: pointSize, color: pointColor },
  { lat: 19.0760, lng: 72.8777, size: pointSize, color: pointColor },
  { lat: 31.2304, lng: 121.4737, size: pointSize, color: pointColor },
  { lat: 39.9042, lng: 116.4074, size: pointSize, color: pointColor },
  { lat: 23.1291, lng: 113.2644, size: pointSize, color: pointColor },
  { lat: 37.5665, lng: 126.9780, size: pointSize, color: pointColor },
  { lat: 43.6532, lng: -79.3832, size: pointSize, color: pointColor },
  { lat: 45.5017, lng: -73.5673, size: pointSize, color: pointColor },
  { lat: 49.2827, lng: -123.1207, size: pointSize, color: pointColor },
  { lat: 59.3293, lng: 18.0686, size: pointSize, color: pointColor },

  { lat: 60.1699, lng: 24.9384, size: pointSize, color: pointColor },
  { lat: 59.9139, lng: 10.7522, size: pointSize, color: pointColor },
  { lat: 55.6761, lng: 12.5683, size: pointSize, color: pointColor },
  { lat: 50.0755, lng: 14.4378, size: pointSize, color: pointColor },
  { lat: 47.4979, lng: 19.0402, size: pointSize, color: pointColor },
  { lat: 52.2297, lng: 21.0122, size: pointSize, color: pointColor },
  { lat: 38.7223, lng: -9.1393, size: pointSize, color: pointColor },
  { lat: 40.4168, lng: -3.7038, size: pointSize, color: pointColor },
  { lat: 45.7640, lng: 4.8357, size: pointSize, color: pointColor },

  { lat: 30.0444, lng: 31.2357, size: pointSize, color: pointColor },
  { lat: -1.2921, lng: 36.8219, size: pointSize, color: pointColor },
  { lat: -26.2041, lng: 28.0473, size: pointSize, color: pointColor },
  { lat: 6.5244, lng: 3.3792, size: pointSize, color: pointColor },
  { lat: 14.5995, lng: 120.9842, size: pointSize, color: pointColor },
  { lat: -6.2088, lng: 106.8456, size: pointSize, color: pointColor },
  { lat: -36.8485, lng: 174.7633, size: pointSize, color: pointColor },
  { lat: 64.1466, lng: -21.9426, size: pointSize, color: pointColor }
]




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
        // if (d.properties.name === 'Germany') return contriesColorHover
        return contriesColor
      })
      .polygonSideColor(() => contriesColor)
      .polygonStrokeColor(() => borderColor)

      // Pings
      .pointsData(points)                                   // les points posés sur le globe
      .pointColor(d => d.color)                 // permet de modifier les couleurs de tous les points
      .pointRadius(d => d.size)                // permet de modifier les couleurs de tous les points
      .pointAltitude(d => d.altitude || 0.03) // contrôle la hauteur du point

  scene.add(globe)
  let t = 0

  function animatePoints() {
    t += 0.03

    globe.pointRadius(d =>
        d.size * (1 + 0.25 * Math.sin(t))
    )
  }
  function animate() {
    requestAnimationFrame(animate)
    animatePoints()       // OBLIGATOIRE

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
