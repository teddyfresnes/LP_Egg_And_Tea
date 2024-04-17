<script>
    import {Link, link} from "svelte-routing";
    let links = ['viennoiseries', 'patisseries'];
    let logos = ['src/assets/ico_steps_action/viennoiseries.png', 'src/assets/ico_steps_action/patisseries.png'];
</script>

<div class="hero">
    <h2>Choisissez une catégorie qui vous donne envie</h2>
  </div>
<div class="product-container">
    {#each links as link, index}
        <Link to="/{link}" class="product">
            <div class="product-logo"> 
                <img alt='tqt' src={logos[index]} /> 
            </div>
            <div class="product-title"> 
                <h2 class="">{link.charAt(0).toUpperCase() + link.slice(1)}</h2> 
            </div>
        </Link>
    {/each}
</div>