<script>
    import L from 'leaflet';
    import { map } from '../../Store/store';

    export let lat;
    export let lon;
    export let title;
    export let marker;

    marker = L.marker([lat, lon], {
        title: `${title}`
    }).addTo($map);

    marker.bindPopup(`${title}`);
</script>