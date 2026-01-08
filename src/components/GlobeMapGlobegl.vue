<template>
  <div ref="globeContainer" class="globe-container"></div>

  <!-- Tooltip au survol -->
  <div v-if="hoveredPoint" class="tooltip">
    {{ hoveredPoint.name }}
  </div>

  <!-- Profil affiché au clic -->
  <div v-if="selectedUser" class="profile">
    <h3>{{ selectedUser.name }}</h3>
    <p>Lat: {{ selectedUser.lat }}, Lng: {{ selectedUser.lng }}</p>
    <!-- Ajoute ici d'autres infos utilisateur -->
    <button @click="selectedUser = null">Fermer</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Globe from 'globe.gl'

const globeContainer = ref(null)

const countriesColor = '#A8D8FF'
const borderColor = '#2C2F33'
const countriesColorHover = '#6A5ACD'
const oceanColor = '#007BFF'
const pointsColor = '#2C2F33'
const pointsSize = 0.3;

let hoveredCountry = null
const hoveredPoint = ref(null)
const selectedUser = ref(null) // utilisateur sélectionné au clic
let globeInstance = null // référence du globe

function createColorCanvasDataURL(color) {
  const canvas = document.createElement('canvas')
  canvas.width = 1
  canvas.height = 1
  const ctx = canvas.getContext('2d')
  ctx.fillStyle = color
  ctx.fillRect(0, 0, 1, 1)
  return canvas.toDataURL()
}
const globeTexture = createColorCanvasDataURL(oceanColor)

const points = [
  { lat: 48.8566, lng: 2.3522, color: pointsColor, size: pointsSize, name: 'Paris' },
  { lat: 40.7128, lng: -74.0060, color: pointsColor, size: pointsSize, name: 'New York' },
  { lat: 35.6895, lng: 139.6917, color: pointsColor, size: pointsSize, name: 'Tokyo' }
]

onMounted(async () => {
  globeInstance = Globe()(globeContainer.value)
      .globeImageUrl(globeTexture)
      .pointsData(points)
      .pointLat(d => d.lat)
      .pointLng(d => d.lng)
      .pointColor(d => d.color)
      .pointAltitude(0.03)
      .pointRadius(d => d.size)
      .onPointHover(point => {
        // réinitialiser couleur
        points.forEach(p => p.color = pointsColor)
        if(point){
          hoveredPoint.value = point
          point.color = countriesColorHover
        } else {
          hoveredPoint.value = null
        }
        globeInstance.pointsData([...points])
      })
      .onPointClick(point => {
        if(point){
          selectedUser.value = point
          // zoomer sur le point cliqué
          const { lat, lng } = point
          globeInstance.pointOfView(
              {
                lat:lat-10,
                lng,
                altitude: 1
              }
              ,1500)
        }
      })

  const countries = await fetch('/world.geojson').then(res => res.json())
  globeInstance.polygonsData(countries.features)
      .polygonCapColor(d => d === hoveredCountry ? countriesColorHover : countriesColor)
      .polygonSideColor(() => countriesColor)
      .polygonStrokeColor(() => borderColor)
      .polygonAltitude(0.01)
      .onPolygonHover(country => {
        hoveredCountry = country
        globeInstance.polygonsData([...countries.features])
      })
})
</script>

<style>
.globe-container {
  width: 100%;
  height: 100vh;
}

.tooltip {
  position: absolute;
  top: 10px;
  left: 10px;
  background: rgba(0,0,0,0.7);
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  pointer-events: none;
}

.profile {
  position: absolute;
  top: 50px;
  left: 10px;
  background: rgba(255,255,255,0.95);
  color: #333;
  padding: 10px 15px;
  border-radius: 6px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
}
</style>
