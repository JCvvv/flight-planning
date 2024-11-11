<template>
  <div id="mapContainer" class="map-container"></div>
</template>

<script lang="ts">
import { defineComponent, onMounted, watch, PropType } from 'vue';

export default defineComponent({
  props: {
    markers: {
      type: Array as PropType<{ id: number; lat: number; lng: number }[]>,
      required: true,
    },
    polygonPath: {
      type: Array as PropType<[number, number][]>,
      required: true,
    },
  },
  emits: ['addMarker', 'updateMarkerPosition'],
  setup(props, { emit }) {
    let map: any;
    let polygon: any;

    onMounted(() => {
      const satelliteLayer = new AMap.TileLayer.Satellite();
      const roadNet = new AMap.TileLayer.RoadNet();
      map = new AMap.Map('mapContainer', {
        center: [117.140109, 36.667382],
        zoom: 15,
        layers: [
          satelliteLayer,
          roadNet
        ],
      });

      props.markers.forEach(markerData => addMarkerToMap(markerData));

      map.on('click', (e: any) => {
        const markerData = { lat: e.lnglat.getLat(), lng: e.lnglat.getLng() };
        emit('addMarker', markerData);
      });

      drawPolygon(props.polygonPath);
    });

    const addMarkerToMap = (markerData: { id: number; lat: number; lng: number }) => {
      const marker = new AMap.Marker({
        position: [markerData.lng, markerData.lat],
        map: map,
        draggable: true,
        content: '<div style="width:10px;height:10px;background:red;border-radius:50%;"></div>',
      });

      marker.on('dragend', (e: any) => {
        emit('updateMarkerPosition', markerData.id, e.lnglat.getLat(), e.lnglat.getLng());
      });
    };

    const drawPolygon = (path: [number, number][]) => {
      if (polygon) {
        map.remove(polygon);
      }

      if (path.length < 3) return;

      polygon = new AMap.Polygon({
        path,
        strokeColor: '#ffffff',
        strokeWeight: 2,
        fillColor: '#d3a4f9',
        fillOpacity: 0.5,
      });

      map.add(polygon);
      displayDistances(path);
    };

    const displayDistances = (path: [number, number][]) => {
      map.getAllOverlays('text').forEach((overlay: any) => map.remove(overlay));

      for (let i = 0; i < path.length; i++) {
        const start = path[i];
        const end = path[(i + 1) % path.length];
        const distance = AMap.GeometryUtil.distance(start, end);

        const midPoint = [
          (start[0] + end[0]) / 2,
          (start[1] + end[1]) / 2,
        ];

        const text = new AMap.Text({
          text: `${distance.toFixed(2)}米`,
          anchor: 'center',
          draggable: false,
          cursor: 'default',
          style: {
            color: '#000',
            fontSize: '15px',
            backgroundColor: 'transparent',
            border:'0'
          },
          position: midPoint,
        });

        map.add(text);
      }
    };

    watch(
      () => props.polygonPath,
      (newPath) => {
        drawPolygon(newPath);
      },
      { deep: true }
    );

    watch(
      () => props.markers,
      (newMarkers) => {
        map.clearMap();
        newMarkers.forEach(addMarkerToMap);
      },
      { deep: true }
    );

    return {};
  },
});
</script>

<style scoped>
.map-container {
  width: 100%;
  height: 100%;
}
</style>
