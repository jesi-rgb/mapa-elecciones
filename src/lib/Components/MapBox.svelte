<script>
	import mapboxgl from 'mapbox-gl';
	import { onMount } from 'svelte';
	import { PUBLIC_MAPBOX_PK } from '$env/static/public';
	import Stat from './Stat.svelte';

	mapboxgl.accessToken = PUBLIC_MAPBOX_PK;

	let hoveredPolygonId = null;
	let properties;
	let map;
	let outlineVisibility = false;
	$: console.log(outlineVisibility);

	onMount(async () => {
		map = new mapboxgl.Map({
			container: 'map', // container ID
			// Choose from Mapbox's core styles, or make your own style with Mapbox Studio
			style: 'mapbox://styles/jesi-rgb/clkky2llt00g301qy93c37rls', // style URL
			center: [-3.7, 40.46], // starting position
			zoom: 5 // starting zoom
		});

		map.on('load', () => {
			map.addSource('elecciones', {
				type: 'geojson',
				data: '/SECC_CE_20230101.json',
				tolerance: 0.8,
				generateId: true
			});

			map.addLayer({
				id: 'elecciones-outline',
				type: 'line',
				source: 'elecciones',
				layout: {
					visibility: outlineVisibility ? 'visible' : 'none'
				},
				paint: {
					'line-color': '#000',
					'line-width': ['case', ['boolean', ['feature-state', 'hover'], false], 3, 0.3],
					'line-opacity': ['case', ['boolean', ['feature-state', 'hover'], false], 1, 0.3]
				}
			});

			map.addLayer({
				id: 'elecciones-fill',
				type: 'fill',
				source: 'elecciones', // reference the data source
				paint: {
					'fill-color': [
						'match',
						['get', 'ganador'],
						'PP',
						'#1E4B8F',
						'PSOE',
						'#E5191F',
						'Sumar',
						'#E7215B',
						'Vox',
						'#7BBA34',
						'EH Bildu',
						'yellow',
						'EAJ-PNV',
						'lightgreen',
						'ERC',
						'salmon',
						'Junts',
						'orange',
						'#ccc'
					],
					'fill-opacity': [
						'case',
						['boolean', ['feature-state', 'hover'], false],
						1,
						['interpolate', ['linear'], ['to-number', ['get', 'pganador'], 50], 0, 0, 100, 1]
					]
				}
			});
		});

		map.on('mousemove', 'elecciones-fill', (e) => {
			if (e.features.length > 0) {
				if (hoveredPolygonId !== null) {
					map.setFeatureState({ source: 'elecciones', id: hoveredPolygonId }, { hover: false });
				}

				hoveredPolygonId = e.features[0].id;
				properties = e.features[0].properties;

				map.setFeatureState({ source: 'elecciones', id: hoveredPolygonId }, { hover: true });
			}
		});

		map.on('mouseleave', 'elecciones-fill', () => {
			if (hoveredPolygonId !== null) {
				map.setFeatureState({ source: 'elecciones', id: hoveredPolygonId }, { hover: false });
			}
			hoveredPolygonId = null;
		});
	});
</script>

<div class="flex flex-col lg:flex-row items-start w-[95%] lg:w-[80%] mx-auto my-10 justify-between">
	<div
		id="map"
		class="w-full mx-auto lg:mx-0 lg:w-3/4 h-[400px] lg:h-[800px] border-2 border-black mb-5 lg:mb-0"
	/>
	<div class="flex flex-col text-2xl space-y-4 w-[95%] lg:w-1/4 lg:pl-2 tabular-nums">
		{#if properties}
			<Stat title="Provincia" value={properties['NPRO']} />
			<Stat title="Municipio" value={properties['NMUN']} />
			<div class="text-base">
				<Stat title="Distrito" value={properties['CDIS']} />
				<Stat title="Sección" value={properties['CSEC']} />
			</div>
			<hr class="my-3" />
			<Stat title="Ganador" value={properties['ganador']} />
			<Stat title="%" value={properties['pganador']} />
			<Stat title="Abstención" value={properties['abstencion'] + '%'} />
		{:else}
			<div>Selecciona una sección para analizar sus datos</div>
		{/if}
	</div>
</div>
