<script>
    import L from 'leaflet'
    import geodesic from 'leaflet.geodesic';
    import { translate } from 'svelte-intl';
    import { map } from '../../Store/store';
    
    export let latStart;
    export let lonStart;
    export let latEnd;
    export let lonEnd;
    export let titleFrom;
    export let titleTo;
    export let line;
    export let countryCode;


    line = L.geodesic([[latStart, lonStart], [latEnd, lonEnd]], {
			weight: 5,
			steps: 10
    }).addTo($map);
    
    const distance = line.distance([latStart, lonStart], [latEnd, lonEnd]);

    line.bindPopup(`${$translate('line.popup.from')} ${titleFrom} ${$translate('line.popup.to')} ${titleTo}, ${$translate(countryCode)}. ${$translate('line.popup.distance')} - ${Math.floor(distance/1000)} ${$translate('line.popup.km')}`);
</script>