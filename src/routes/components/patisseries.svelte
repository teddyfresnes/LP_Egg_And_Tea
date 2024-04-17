<script>
	import { Link } from "svelte-routing";
	import patisseries from '../../assets/patisseries.json';
	let logo = 'src/assets/ico_steps_action/patisseries.png';
</script>

<!-- Afficher les patisseries disponibles -->
<div class="hero">
    <h2>Liste de toutes les patisseries</h2>
</div>
<div class="product-container">
	{#each patisseries as patisserie}
		<Link to={'/affichage/'+patisserie.name} class="product">
			<div class="product-logo"> 
				<img alt='tqt' src={logo} /> 
			</div>
			<div class="product-title"> 
				<h2>{patisserie.name}</h2> 
			</div>
		</Link>
	{/each}
</div>