<script>
  import { onMount } from 'svelte';
  import L from 'leaflet';
  import { map } from '../../Store/store';
  
  onMount(async () => {
		map.update(map => map = L.map("map", {
			zoomControl: false
		}))

		L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
			attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
		}).addTo($map);

		$map.setView([45.00, 5.00], 3);
		
		L.control.zoom({
			position: 'topright'
		}).addTo($map);
	});
</script>

<div id="map" />

<style>
  #map {
		height: 100%;
	}
</style>