<script>
import {
		Button,
		Collapse,
		Icon,
		Field,
		Tabs,
		Tab,
		Tag,
		Select
  } from 'svelma';
  import L from 'leaflet';
	import geodesic from 'leaflet.geodesic';
	import _ from 'lodash';
	import { translate } from 'svelte-intl';
	import functions from 'firebase-functions';
  import AutoComplete from '../Autocomplite/AutoComplite.svelte';
  import {
    suggestions,    
    data,
    numberOfRoutes,
		departureAirportData,
		arrivalAirportData,
    selectedDepartureAirport,
    selectedArrivalAirport,
    markerArrival,
		markerDeparture,
		line,
		countryCodeData,
		numberOfCountryes,
		filteredData,
		selected,
		map,
		selectedCountryes,
		loadedArrivalCode,
		loadedDepartureCode,
	} from '../../Store/store';

  async function loadDataByTerm(term) {
		const res = await fetch(`https://aerodatabox.p.rapidapi.com/airports/search/term?limit=10&q=${term}`, {
			"method": "GET",
			"headers": {
				"x-rapidapi-host": "aerodatabox.p.rapidapi.com",
				"x-rapidapi-key": `${functions.config().api.key}`
			}
		});
		let suggestionsData = await res.json();

		return _.map(suggestionsData.items, o => {
				return {key: o.icao, value: o.name}
			}
		);
	}

	async function loadDepartureRoutes() {
		const res = await fetch(
			`https://aerodatabox.p.rapidapi.com/airports/icao/${$selectedDepartureAirport.key}/stats/routes/daily`, {
					"method": "GET",
					"headers": {
						"x-rapidapi-host": "aerodatabox.p.rapidapi.com",
						"x-rapidapi-key": `${functions.config().api.key}`
					}
				}
			);
		let routesData = await res.json();

		data.update(d => d = routesData);
		
		numberOfRoutes.update(n => n = routesData.routes.length);
	}

	async function loadDepartureAirportData() {
		const departureAirportInfo = await fetch(
			`https://aerodatabox.p.rapidapi.com/airports/icao/${$selectedDepartureAirport.key}`, {
				"method": "GET",
				"headers": {
					"x-rapidapi-host": "aerodatabox.p.rapidapi.com",
					"x-rapidapi-key": `${functions.config().api.key}`
				}
			}
		);
		let departureInfo = await departureAirportInfo.json();

		departureAirportData.update(d => d = departureInfo);

		loadedDepartureCode.update(l => l = $selectedDepartureAirport.key);
	}

	async function loadArrivalAirportData() {
		const arrivalAirportInfo = await fetch(
			`https://aerodatabox.p.rapidapi.com/airports/icao/${$selectedArrivalAirport.key}`, {
				"method": "GET",
				"headers": {
					"x-rapidapi-host": "aerodatabox.p.rapidapi.com",
					"x-rapidapi-key": `${functions.config().api.key}`
				}
			}
		);
		let arrivalInfo = await arrivalAirportInfo.json();
		
		arrivalAirportData.update(a => a = arrivalInfo);

		loadedArrivalCode.update(l => l = $selectedArrivalAirport.key);
	}

	async function loadData() {
		if (
			$selectedDepartureAirport.key &&
			$selectedDepartureAirport.key.length === 4
		) {
			if ($loadedDepartureCode !== $selectedDepartureAirport.key) {
				if (
					$map.hasLayer($markerArrival) ||
					$map.hasLayer($markerDeparture) ||
					$map.hasLayer($line)
				) {
					$markerArrival.remove();
					$markerDeparture.remove();
					$line.remove();
				}
				
				await loadDepartureRoutes();

				await loadDepartureAirportData();

			} else if ($loadedDepartureCode === $selectedDepartureAirport.key) {
				if (
					$map.hasLayer($markerArrival) ||
					$map.hasLayer($markerDeparture) ||
					$map.hasLayer($line)
				) {
					$markerArrival.remove();
					$markerDeparture.remove();
					$line.remove();
				}
			}

		} else if (!($selectedDepartureAirport.key)) {
			data.set([]);
			departureAirportData.set([]);
			numberOfRoutes.set(0);
		} else {
			console.log('Wrong departure ICAO code.');
		}

		if (
			$selectedArrivalAirport.key &&
			$selectedArrivalAirport.key.length === 4
		) {

			if ($loadedArrivalCode !== $selectedArrivalAirport.key) {

				await loadArrivalAirportData();
				
			} else if ($loadedArrivalCode === $selectedArrivalAirport.key) {
				if (
					$map.hasLayer($markerArrival) ||
					$map.hasLayer($markerDeparture) ||
					$map.hasLayer($line)
				) {
					$markerArrival.remove();
					$markerDeparture.remove();
					$line.remove();
				}
			}
		} else if (!($selectedArrivalAirport.key)) {
			arrivalAirportData.set([]);
		} else {
			console.log('Wrong arrival ICAO code.');
		}
		
		extractingCountryCodeData();
		sortingAndFilteringData();
	}

	function extractingCountryCodeData() {
		countryCodeData.update(c => c = _($data.routes || [])
			.map((r) => r.destination.countryCode)
			.uniq()
			.orderBy()
			.value());
		numberOfCountryes.update(n => n = $countryCodeData.length) ;
	}

	function sortingAndFilteringData() {
		filteredData.update(f => f = _($data.routes || [])
			.groupBy('destination.countryCode')
			.transform(function(result, value, key) {
				result[key] = _.orderBy((value || []), ['destination.municipalityName'], ['asc']);
			}, [])
			.value());
	}
  
  function handleRouteSelect(route, i, j) {
		let {
			destination: {
				countryCode,
				icao,
				location: {
					lat,
					lon
				}
			}
		} = route;

		if (
			!($map.hasLayer($markerDeparture)) &&
			!($map.hasLayer($markerArrival)) &&
			!($map.hasLayer($line))
		) {

			selected.update(s => s = String(i) + String(j));

			drawDepartureMarker();

			drawArrivalMarker(
				icao,
				lat,
				lon,
			);

			drawPolyline(
				countryCode,
				icao,
				lat,
				lon
			);

		}  else if ( $selected !== String(i) + String(j)) {

			$markerDeparture.remove();
			$markerArrival.remove();
			$line.remove();

			selected.update(s => s = String(i) + String(j));

			drawDepartureMarker();

			drawArrivalMarker(
				icao,
				lat,
				lon,
			);

			drawPolyline(
				countryCode,
				icao,
				lat,
				lon
			);

		} else if ($selected === String(i) + String(j)) {
			$markerDeparture.remove();
			$markerArrival.remove();
			$line.remove();
		}
  }
  
  function drawDepartureMarker() {
		markerDeparture.update(m => m = L.marker($departureAirportData.location, {
				title: `${$departureAirportData.shortName}, ${$departureAirportData.municipalityName}`
			}).addTo($map));

			$markerDeparture.bindPopup(`${$departureAirportData.shortName}, ${$departureAirportData.municipalityName}`);
	}

	function drawArrivalMarker(icao, lat, lon) {
		markerArrival.update(m => m = L.marker([lat, lon], {
				title: `${$translate(icao)}`
			}).addTo($map));

			$markerArrival.bindPopup(`${$translate(icao)}`);
	}

	function drawPolyline(countryCode, icao, lat, lon) {
		line.update(l => l = L.geodesic([[$departureAirportData.location.lat, $departureAirportData.location.lon], [lat, lon]], {
			weight: 5,
			steps: 10
		}).addTo($map))

		const distance = $line.distance([$departureAirportData.location.lat, $departureAirportData.location.lon], [lat, lon]);

		$line.bindPopup(
			`${$translate('line.popup.from')} ${$departureAirportData.shortName}, ${$departureAirportData.municipalityName} ${$translate('line.popup.to')}
			${$translate(icao)}, ${$translate(countryCode)}. ${$translate('line.popup.distance')} - ${Math.floor(distance/1000)} ${$translate('line.popup.km')}`
		);
	}
</script>

<main>
  <Collapse animation="fade">
  <button
    class="button is-primary"
    slot="trigger"
  >	  
    <Icon icon="bars"/>
  </button>
  <div class="menu">
    <div>
    <Tabs style="is-boxed">
      <Tab
        label={$translate('tab.search.label')}
        icon="search"
      >
        <Field label={$translate('tab.search.input.departure.label')}>
          <AutoComplete
            items={$suggestions}
            asyncFunc={loadDataByTerm}
            bind:value={$selectedDepartureAirport}
            id="departure"
          />
        </Field>
        <Field label={$translate('tab.search.input.arrival.label')}>
          <AutoComplete
            items={$suggestions}
            asyncFunc={loadDataByTerm}
            bind:value={$selectedArrivalAirport}
            id="arrival"
          />		
        </Field>
        <Button
          type="is-primary"
          on:click={loadData}
        >
          <Icon icon="search"/>
          {$translate('tab.search.button.search')}
        </Button>
        {#if
          !($arrivalAirportData.municipalityName) &&
          !($selectedArrivalAirport.key) &&
          $departureAirportData.municipalityName &&
					$selectedDepartureAirport.key &&
					$loadedDepartureCode === $selectedDepartureAirport.key
        }
          {#if {$numberOfRoutes} !== 0}
            <Tag type='is-success is-light is-large'>
              {$translate('tab.search.tag.start')}
              {$numberOfRoutes}
              {$translate('tab.search.tag.end')}
            </Tag>
          {/if}
        {/if}
      </Tab>
      <Tab
        label={$translate('tab.filters.label')}
        icon="filter"
      >
				{#if
					!($arrivalAirportData.municipalityName) &&
					!($selectedArrivalAirport.key) &&
					$departureAirportData.municipalityName &&
					$selectedDepartureAirport.key &&
					$loadedDepartureCode === $selectedDepartureAirport.key
				}
          {#if $numberOfCountryes !== 0}
            <h6 class="title is-6">
              {$translate('tab.filters.select.title')}
						</h6>
						<div class="menu-list">
							<Field>
								<Select
									multiple
									nativeSize="6"
									bind:selected={$selectedCountryes}
								>
									{#each ($countryCodeData || []) as countryCode}
										{#if countryCode}
											<option value={countryCode}>
												{$translate(countryCode)}
											</option>
										{/if}
									{/each}
								</Select>
							</Field>
							<Button
								type="is-danger"
								on:click={() => selectedCountryes.set([])}
							>
							{$translate('tab.filters.button.clear')}
							</Button>
						</div>
          {/if}
        {/if}
      </Tab>
      <Tab
        label={$translate('tab.routes.label')}
        icon="route"
      >					
        <aside class="menu">
          <ul class="menu-list">
            {#if
							!($arrivalAirportData.municipalityName) &&
							!($selectedArrivalAirport.key) &&
							$departureAirportData.municipalityName &&
							$selectedDepartureAirport.key && 
							$loadedDepartureCode === $selectedDepartureAirport.key
						}
              {#if $selectedCountryes.length === 0}
                {#each ($countryCodeData || []) as countryCode, j }
                  {#if countryCode}
                    <p class="menu-label">
                      {$translate(countryCode)}
                    </p>
                  {/if}								
                  {#each ($filteredData[countryCode] || []) as route, i}
                    {#if route.destination.municipalityName}
                      <li>
                        <a on:click={() => handleRouteSelect(route, i, j)}>
                          {$translate(route.destination.icao)}
                        </a>											
                      </li>
                    {/if}
                  {/each}
                {/each}
              {:else if $selectedCountryes.length !== 0}
                {#each ($selectedCountryes || []) as countryCode, j }
                  {#if countryCode}
                    <p class="menu-label">
                      {$translate(countryCode)}
                    </p>
                  {/if}								
                  {#each ($filteredData[countryCode] || []) as route, i}
                    {#if route.destination.municipalityName}
                      <li>
                        <a on:click={() => handleRouteSelect(route, i, j)}>
                          {$translate(route.destination.icao)}
                        </a>											
                      </li>
                    {/if}
                  {/each}
                {/each}
              {/if}
            {/if}
          </ul>
        </aside>
      </Tab>				
    </Tabs>			
    </div>
  </div>
</Collapse>
</main>


<style>
  .menu {
		background-color: white;
		border-radius: 4px;
		padding: 10px;		
	}

	.menu-list {
		max-height: 50vh;
		overflow:hidden;
		overflow-y:scroll;
	}

  main {		
		position: absolute;
		top: 0px;
		left: 0px;
		z-index: 400;
		padding: 1em;
		max-width: 360px;
		margin: 0;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

	@media (min-height: 500px) {
		.menu-list {
			max-height: 70vh;
		}
	}
</style>