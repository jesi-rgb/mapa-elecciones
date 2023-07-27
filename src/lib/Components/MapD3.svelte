<script>
	import { onMount } from 'svelte';
	import * as topojson from 'topojson-client';
	import { geoPath, geoMercator } from 'd3-geo';
	import { draw } from 'svelte/transition';

	let secciones, path, mercator;
	let loading = true;
	onMount(async () => {
		const spain = await fetch('/elecciones-topojson.json').then((d) => d.json());

		mercator = geoMercator({ scale: 2000, translate: [-3.7, 40.46] });
		path = geoPath(mercator);

		secciones = topojson.feature(spain, spain.objects['SECC_CE_20230101']).features;
		console.log(secciones[2], path(secciones[2]));
		loading = false;
	});
</script>

<div>Mapa</div>

{#if loading}
	<div>loading...</div>
{:else}
	<div class="border border-black w-[500px] h-[500px] mx-auto">
		<svg width="500" height="500">
			<g fill="grey" stroke="black">
				{#each secciones as feature, i}
					<path d={path(feature)} class="seccion" />
				{/each}
			</g></svg
		>
	</div>
{/if}
