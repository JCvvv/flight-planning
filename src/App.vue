<template>
  <div class="app-container">
    <Sidebar @switchMap="switchMap" />
    <component
      :is="currentMapComponent"
      class="map-container"
      :markers="markers"
      :polygonPath="polygonPath"
      @addMarker="addMarker"
      @updateMarkerPosition="updateMarkerPosition"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, computed } from 'vue';
import Sidebar from './components/Sidebar.vue';
import SatelliteMap from './views/SatelliteMap.vue';
import StandardMap from './views/StandardMap.vue';
import BuildingsMap from './views/BuildingsMap.vue';

// 凸包算法，返回点集的最小凸包路径
const computeConvexHull = (points: { lat: number; lng: number }[]): [number, number][] => {
  if (points.length < 3) return [];
  points = points.slice().sort((a, b) => a.lng - b.lng || a.lat - b.lat);

  const cross = (o: any, a: any, b: any) => (a.lng - o.lng) * (b.lat - o.lat) - (a.lat - o.lat) * (b.lng - o.lng);
  const lower = [];
  for (const point of points) {
    while (lower.length >= 2 && cross(lower[lower.length - 2], lower[lower.length - 1], point) <= 0) {
      lower.pop();
    }
    lower.push(point);
  }

  const upper = [];
  for (const point of points.reverse()) {
    while (upper.length >= 2 && cross(upper[upper.length - 2], upper[upper.length - 1], point) <= 0) {
      upper.pop();
    }
    upper.push(point);
  }

  lower.pop();
  upper.pop();
  return [...lower, ...upper].map(p => [p.lng, p.lat]);
};

export default defineComponent({
  components: {
    Sidebar,
    SatelliteMap,
    StandardMap,
    BuildingsMap,
  },
  setup() {
    const currentMapComponent = ref('SatelliteMap');
    const markers = reactive([] as { id: number; lat: number; lng: number }[]);
    let markerIdCounter = 0;

    const addMarker = (marker: { lat: number; lng: number }) => {
      markers.push({ id: markerIdCounter++, ...marker });
    };

    const updateMarkerPosition = (markerId: number, newLat: number, newLng: number) => {
      const marker = markers.find(marker => marker.id === markerId);
      if (marker) {
        marker.lat = newLat;
        marker.lng = newLng;
      }
    };

    const polygonPath = computed(() => {
      return markers.length >= 3 ? computeConvexHull(markers) : [];
    });

    const switchMap = (mapType: string) => {
      currentMapComponent.value = mapType;
    };

    return {
      currentMapComponent,
      markers,
      addMarker,
      updateMarkerPosition,
      polygonPath,
      switchMap,
    };
  },
});
</script>

<style>
.app-container {
  display: flex;
  width: 100vw;
  height: 100vh;
}
.map-container {
  flex: 1;
  height: 100vh;
}
</style>
