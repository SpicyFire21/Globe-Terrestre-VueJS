<template>
  <div ref="globeContainer" class="globe-container"></div>

  <!-- Tooltip lié au point -->
  <div
      v-if="hoveredPoint"
      class="tooltip"
      :style="{ top: tooltip.y + 'px', left: tooltip.x + 'px' }"
  >
    {{ hoveredPoint.name }}
  </div>

  <!-- Profil utilisateur lié au point -->
  <div
      v-if="selectedUser"
      class="profile"
      :style="{ top: profilePos.y + 'px', left: profilePos.x + 'px' }"
  >
    <h3>{{ selectedUser.name }}</h3>
    <p>Lat: {{ selectedUser.lat }}</p>
    <p>Lng: {{ selectedUser.lng }}</p>
    <button @click="selectedUser = null">Fermer</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Globe from 'globe.gl'

const globeContainer = ref(null)

const hoveredPoint = ref(null)
const selectedUser = ref(null)

const tooltip = ref({ x: 0, y: 0 })
const profilePos = ref({ x: 0, y: 0 })

let globeInstance = null
let hoveredCountry = null

const countriesColor = '#A8D8FF'
const borderColor = '#2C2F33'
const countriesColorHover = '#6A5ACD'
const oceanColor = '#007BFF'
const pointsColor = '#2C2F33'
const pointsSize = 0.3

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
  { lat: 48.8566, lng: 2.3522, name: 'Paris'  },
  { lat: 40.7128, lng: -74.0060, name: 'New York'  },
  { lat: 35.6895, lng: 139.6917, name: 'Tokyo' }
]

onMounted(async () => {
  globeInstance = Globe()(globeContainer.value)
      .globeImageUrl(globeTexture)
      .pointsData(points)
      .pointLat(d => d.lat)
      .pointLng(d => d.lng)
      .pointColor(d => d.color =pointsColor)
      .pointRadius(d => d.size = pointsSize)
      .pointAltitude(0.03)

      // Hover point
      .onPointHover(point => {
        points.forEach(p => (p.color = pointsColor))

        if (point) {
          hoveredPoint.value = point
          point.color = countriesColorHover
        } else {
          hoveredPoint.value = null
        }

        globeInstance.pointsData([...points])
      })

      // Click point
      .onPointClick(point => {
        if (!point) return

        selectedUser.value = point

        globeInstance.pointOfView(
            {
              lat: point.lat - 8,
              lng: point.lng,
              altitude: 1.1
            },
            1200
        )
      })

  const countries = await fetch('/world.geojson').then(r => r.json())

  globeInstance
      .polygonsData(countries.features)
      .polygonCapColor(d => (d === hoveredCountry ? countriesColorHover : countriesColor))
      .polygonSideColor(() => countriesColor)
      .polygonStrokeColor(() => borderColor)
      .polygonAltitude(0.01)
      .onPolygonHover(country => {
        hoveredCountry = country
        globeInstance.polygonsData([...countries.features])
      })

  // Synchronisation écran <-> globe (OBLIGATOIRE)
  function updateOverlayPositions() {
    if (hoveredPoint.value) {
      const { x, y } = globeInstance.getScreenCoords(
          hoveredPoint.value.lat,
          hoveredPoint.value.lng
      )
      tooltip.value = { x: x + 12, y: y + 12 }
    }

    if (selectedUser.value) {
      const { x, y } = globeInstance.getScreenCoords(
          selectedUser.value.lat,
          selectedUser.value.lng
      )
      profilePos.value = { x: x + 2, y: y + 2 }
    }

    requestAnimationFrame(updateOverlayPositions)
  }

  updateOverlayPositions()
})
</script>

<style>
.globe-container {
  width: 100%;
  height: 100vh;
}

.tooltip {
  position: fixed;
  background: rgba(0, 0, 0, 0.75);
  color: #fff;
  padding: 6px 10px;
  border-radius: 4px;
  font-size: 12px;
  pointer-events: none;
  z-index: 1000;
}

.profile {
  position: fixed;
  background: #F5F5F5;
  border: #6A5ACD solid 2px;
  color: #2C2F33;
  padding: 12px 16px;
  border-radius: 6px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
  z-index: 1000;
}
</style>
