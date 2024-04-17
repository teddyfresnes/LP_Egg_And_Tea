<script>
	import { Link } from "svelte-routing";
	import viennoiseries from '../../assets/viennoiseries.json';
	let logo = 'src/assets/ico_steps_action/viennoiseries.png';
</script>

<!-- Afficher les viennoiseries disponibles -->

<div class="hero">
    <h2>Liste de toutes les viennoiseries</h2>
</div>
<div class="product-container">
	{#each viennoiseries as viennoiserie}
		<Link to={'/affichage/'+viennoiserie.name} class="product">
			<div class="product-logo"> 
				<img alt='tqt' src={logo} /> 
			</div>
			<div class="product-title"> 
				<h2>{viennoiserie.name}</h2> 
			</div>
		</Link>
	{/each}
</div>
