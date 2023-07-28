<script>
	import '../app.postcss';
	import Footer from '../lib/Components/Footer.svelte';
</script>

<div class="w-[95%] lg:w-3/4 mx-auto">
	<slot />
	<Footer />
</div>
