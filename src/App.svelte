<script context="module">
	import { locale, translations } from 'svelte-intl';
	import translationUi from './Translations/translationsUi';
	import translationIcaoRu from './Translations/translationIcaoRu';
	import translationIcaoEn from './Translations/translationIcaoEn';
	import translationsCountryCode from './Translations/translationsCountryCode';
  
	translations.update(translationUi);
	translations.update(translationIcaoRu);
	translations.update(translationIcaoEn);
	translations.update(translationsCountryCode);

  locale.set(window.navigator.language);
</script>


<script>
	import 'bulma/css/bulma.css';
	import '@fortawesome/fontawesome-free/css/all.css';
	import Marker from './Components/Marker/Marker.svelte';
	import Polyline from './Components/Polyline/Polyline.svelte';
	import Map from './Components/Map/Map.svelte';
	import Menu from './Components/Menu/Menu.svelte';
	import {
		map,
		line,
		arrivalAirportData,
		departureAirportData,
		selectedArrivalAirport,
		markerArrival,
		markerDeparture,
 } from './Store/store';
</script>

{#if
	$arrivalAirportData.municipalityName &&
	$selectedArrivalAirport.key
}	
	<Marker
		bind:lat={$arrivalAirportData.location.lat}
		bind:lon={$arrivalAirportData.location.lon}
		title={`${$arrivalAirportData.shortName}, ${$arrivalAirportData.municipalityName}`}
		bind:marker={$markerArrival}
	/>
	<Marker
		bind:lat={$departureAirportData.location.lat}
		bind:lon={$departureAirportData.location.lon}
		title={`${$departureAirportData.shortName}, ${$departureAirportData.municipalityName}`}
		bind:marker={$markerDeparture}
	/>
	<Polyline
		latStart={$departureAirportData.location.lat}
		lonStart={$departureAirportData.location.lon}
		latEnd={$arrivalAirportData.location.lat}
		lonEnd={$arrivalAirportData.location.lon}
		titleFrom={`${$departureAirportData.shortName}, ${$departureAirportData.municipalityName}`}
		titleTo={`${$arrivalAirportData.shortName}, ${$arrivalAirportData.municipalityName}`}
		countryCode={$arrivalAirportData.country.code}
		bind:line={$line}
	/>
{/if}

<Map />

<Menu />