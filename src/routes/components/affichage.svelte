<!-- Afficher les données de la viennoiserie ou de la patisserie choisie -->
<script>
	import Timer from './timer.svelte';
	import patisseries from '../../assets/patisseries.json';
	import viennoiseries from '../../assets/viennoiseries.json';
	
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';

	import Ajouter from '../../assets/ico_steps_action/Ajouter.png';
	import Placer from '../../assets/ico_steps_action/Ajouter.png';
	import Battre from '../../assets/ico_steps_action/Battre.png';
	import Beurrer from '../../assets/ico_steps_action/Beurrer.png';
	import Bouillir from '../../assets/ico_steps_action/Bouillir.png';
	import Couvrir from '../../assets/ico_steps_action/Couvrir.png';
	import Cuire from '../../assets/ico_steps_action/Cuire.png';
	import Decouper from '../../assets/ico_steps_action/Decouper.png';
	import Default from '../../assets/ico_steps_action/default.png';
	import Derouler from '../../assets/ico_steps_action/Derouler.png';
	import Diviser from '../../assets/ico_steps_action/Diviser.png';
	import Dorer from '../../assets/ico_steps_action/Dorer.png';
	import Enfourner from '../../assets/ico_steps_action/Enfourner.png';
	import Fariner from '../../assets/ico_steps_action/Fariner.png';
	import Fondre from '../../assets/ico_steps_action/Fondre.png';
	import Humidifier from '../../assets/ico_steps_action/Humidifier.png';
	import Melanger from '../../assets/ico_steps_action/Melanger.png';
	import Piquer from '../../assets/ico_steps_action/Piquer.png';
	import Prechauffer from '../../assets/ico_steps_action/Prechauffer.png';
	import Pétrir from '../../assets/ico_steps_action/Pétrir.png';
	import Recouvrir from '../../assets/ico_steps_action/Recouvrir.png';
	import Refroidir from '../../assets/ico_steps_action/Refroidir.png';
	import Reposer from '../../assets/ico_steps_action/Reposer.png';
	import Rouler from '../../assets/ico_steps_action/Rouler.png';
	import Verser from '../../assets/ico_steps_action/Verser.png';


	const step_images = { // on peut egalement utiliser eval, mais pas sécurisé
		"Ajouter.png": Ajouter,
		"Placer.png": Placer,
		"Battre.png": Battre,
		"Beurrer.png": Beurrer,
		"Bouillir.png": Bouillir,
		"Couvrir.png": Couvrir,
		"Cuire.png": Cuire,
		"Decouper.png": Decouper,
		"default.png": Default,
		"Derouler.png": Derouler,
		"Diviser.png": Diviser,
		"Dorer.png": Dorer,
		"Enfourner.png": Enfourner,
		"Fariner.png": Fariner,
		"Fondre.png": Fondre,
		"Humidifier.png": Humidifier,
		"Melanger.png": Melanger,
		"Piquer.png": Piquer,
		"Prechauffer.png": Prechauffer,
		"Pétrir.png": Pétrir,
		"Recouvrir.png": Recouvrir,
		"Refroidir.png": Refroidir,
		"Reposer.png": Reposer,
		"Rouler.png": Rouler,
		"Verser.png": Verser,
	};


	// recuperer l'url
	const url = window.location.pathname;
	//alert(url);

	// pop pour recuperer le dernier élement (donc le nom de la recette)
	const url_nom_recette = url.split('/').pop();
	const nom_recette = decodeURIComponent(url_nom_recette);

	// parcours la liste jusqu'à trouver la recette que l'on veut
	let recette = patisseries.find(recipe => recipe.name === nom_recette) ||
	viennoiseries.find(recipe => recipe.name === nom_recette);


	// occurence de l'étape pour le timer et l'affichage
	let step_index = 0;

	// compte a rebourd de la premiere etape
	let countdown = recette.steps[step_index].time;


	// le temps est finit
	function handleTimerEnd() {
		// gestion du li actif
		const activeStepElement = document.querySelector('.activeStep');
		activeStepElement.classList.add('fade-out'); // transition

        if (step_index+1 < recette.steps.length) { // check si l'étape existe
            step_index += 1; // index +1 = étape +1
            countdown = recette.steps[step_index].time;

			
			// Une fois la transition faite on supprimera le li
			activeStepElement.addEventListener('transitionend', () => {
				activeStepElement.remove();
			});

			// On affiche que l'on passe à l'étape suivante
            console.log("on passe à l'étape ",step_index+1);
        }
        else{ // Si il n'y a plus d'étapes
            countdown = -1;
            console.log("fin des étapes");
        }
    }


	// Faire dérouler les ingrédients 
	let showIngredients = false;

    function toggleIngredients() {
        showIngredients = !showIngredients;
    }

	// Non-interactive elements with an on:click event must be accompanied by a keyboard event handler
    function handleKeydown(event) {
        if (event.key === 'Enter' || event.key === ' ') {
            toggleIngredients();
        }
    }

    onMount(() => {
        // Afficher les ingrédients au chargement de la page
        showIngredients = true;
    });
</script>


{#if recette}
	<!-- affichage recette -->
	<div class="hero">
		<h2>~ {recette.name} ~</h2>
	</div>

	<div class="crafting_product">
		<div class="ingredients {showIngredients ? 'open' : ''}">
			<div class="ingredients-toggle" role="button" tabindex="0" on:click={toggleIngredients} on:keydown={handleKeydown}>
				<h2>Ingredients :</h2>
				<p class="ingredients-toggle-defiler">{showIngredients ? '-' : '+'}</p>
			</div>
			<ul class="ingredients-list">
				{#each Object.entries(recette.ingredients) as [ingredient, quantity]}
					<li>{ingredient.charAt(0).toUpperCase() + ingredient.slice(1)}: {quantity}</li>
				{/each}
			</ul>
		</div>
		
		<!-- timer -->
		<div class="timer_list">
			<div class="timer_wrapper">
				<!-- le timer envoie l'event "timerEnd" quand le minuteur est à 0 -->
				<Timer countdown={countdown} on:timerEnd={handleTimerEnd}/>
			</div>
	
			<!-- etapes de la recette -->
			<ol class="liste_recettes" id="liste_recettes">
				{#each recette.steps as step, i}
					<li class:activeStep={i===step_index}>
						<!-- eval pour transformer les string en composent appelé, cependant vulnérable, pour la sécurité c'est préférable de faire des listes... -->
						<img src={step_images[`${step.type}.png`]} alt={step.type}/>
						<p>{step.description}</p>
					</li>
				{/each}
			</ol>
		</div>
	</div>

	{:else}
		<!-- prevenir si la recette n'a pas été trouvé -->
		<p>la recette n'existe pas</p>
{/if}