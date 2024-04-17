<!-- App.svelte -->
<script>
	import { Router, Link, Route } from "svelte-routing";
	import Accueil from './routes/components/accueil.svelte';
	import Viennoiseries from './routes/components/viennoiseries.svelte'
	import Patisseries from './routes/components/patisseries.svelte'
	import Affichage from './routes/components/affichage.svelte'

	export let url = "";
</script>


<Router {url}>
  <nav>
    <Link to="/" class='link'> Accueil </Link>
  </nav>

  <!-- Gestion des routes -->
  <div>
    <Route path="/">
      <Accueil />
    </Route>

    <!-- Si on clique sur le bouton "Voir les Viennoiseries" -->
    <Route path="/viennoiseries">
      <Viennoiseries />
    </Route>

    <!-- Si on clique sur le bouton "Voir les Patisseries" -->
    <Route path="/patisseries">
      <Patisseries />
    </Route>

	  <!-- si on clique sur le bouton du nom d'une recette -->
	  <Route path="/affichage/:nom_recette">
		  <Affichage />
	  </Route>
  </div>
</Router>