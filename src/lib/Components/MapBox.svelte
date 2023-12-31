<script>
	import mapboxgl from 'mapbox-gl';
	import { onMount } from 'svelte';
	import { PUBLIC_MAPBOX_PK } from '$env/static/public';
	import Stat from './Stat.svelte';

	mapboxgl.accessToken = PUBLIC_MAPBOX_PK;

	let hoveredPolygonId = null;
	let properties;

	$: if (hoveredPolygonId == null) {
		properties = undefined;
	}

	let map;

	function showElectionData() {
		map.setLayoutProperty('elecciones-fill', 'visibility', 'visible');
		map.setLayoutProperty('renta-fill', 'visibility', 'none');
	}
	function showIncomeData() {
		map.setLayoutProperty('elecciones-fill', 'visibility', 'none');
		map.setLayoutProperty('renta-fill', 'visibility', 'visible');
	}

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
				paint: {
					'line-color': '#000',
					'line-width': ['case', ['boolean', ['feature-state', 'hover'], false], 5, 0.3],
					'line-opacity': ['case', ['boolean', ['feature-state', 'hover'], false], 1, 0.1]
				}
			});

			map.addLayer({
				id: 'renta-fill',
				type: 'fill',
				source: 'elecciones', // reference the data source
				layout: {
					visibility: 'none'
				},
				paint: {
					'fill-color': [
						'interpolate-hcl',
						['linear'],
						['to-number', ['get', 'renta_mediana'], 0],
						0,
						'#eee',
						5000,
						'tomato',
						25000,
						'Gold',
						40000,
						'steelblue'
					],
					'fill-opacity': 0.9
				}
			});

			map.addLayer({
				id: 'renta-outline',
				type: 'line',
				source: 'elecciones',
				paint: {
					'line-color': '#000',
					'line-width': ['case', ['boolean', ['feature-state', 'hover'], false], 5, 0.3],
					'line-opacity': ['case', ['boolean', ['feature-state', 'hover'], false], 1, 0.1]
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

		map.on('mousemove', 'renta-fill', (e) => {
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

		map.on('mouseleave', 'renta-fill', () => {
			if (hoveredPolygonId !== null) {
				map.setFeatureState({ source: 'elecciones', id: hoveredPolygonId }, { hover: false });
			}
			hoveredPolygonId = null;
		});
	});
</script>

<div class="flex flex-col lg:flex-row items-start my-10 justify-between h-full">
	<div
		id="map"
		class="w-full mx-auto lg:mr-3 lg:w-[60%] h-[400px] lg:h-[600px] border-2 border-black mb-5 lg:mb-0"
	/>
	<div class="flex flex-col text-xl space-y-4 w-full lg:w-[40%] border h-full">
		<div class="flex justify-around space-x-1">
			<button
				class="p-4 font-bold bg-blue-200 hover:bg-blue-300 transition-colors shadow-blue-800/30 shadow-md rounded-md"
				on:click={() => showElectionData()}>Mostrar Votos</button
			>
			<button
				class="p-4 font-bold bg-blue-200 hover:bg-blue-300 transition-colors shadow-blue-800/30 shadow-md rounded-md"
				on:click={() => showIncomeData()}>Mostrar Renta</button
			>
		</div>
		<div class="border h-full flex flex-col space-y-10">
			{#if properties}
				<div>
					<div class="font-bold text-2xl">Datos de provincia</div>
					<div class="grid grid-cols-2">
						<Stat title="Provincia" value={properties['NPRO']} />
						<Stat title="Municipio" value={properties['NMUN']} />
						<Stat title="Distrito" value={properties['CDIS']} />
						<Stat title="Sección" value={properties['CSEC']} />
					</div>
				</div>
				<div>
					<div class="font-bold text-2xl">Datos electorales</div>
					<div class="flex flex-row justify-start space-x-10">
						<Stat title="Ganador" value={properties['ganador']} />
						<Stat title="%" value={properties['pganador']} />
						<Stat title="Abstención" value={properties['abstencion'] + '%'} />
					</div>
				</div>
				<div>
					<div class="font-bold text-2xl">Datos económicos</div>
					<div class="flex flex-row space-x-4">
						<Stat title="Renta Mediana Anual (2018)" value={properties['renta_mediana'] + '€'} />
						<Stat
							title="Renta Mediana Mensual (2018)"
							value={(Number.parseInt(properties['renta_mediana']) / 12).toFixed(2) + '€'}
						/>
					</div>
				</div>
			{:else}
				<div class="text-gray-700 font-bold h-full">
					Selecciona una sección para analizar sus datos
				</div>
			{/if}
		</div>
	</div>
</div>
