<!-- https://svelte.dev/repl/a19df6f013434e01bb990349843af288?version=3.38.2 -->
<script>
	import { createEventDispatcher, onDestroy, onMount } from 'svelte';
	import { tweened } from 'svelte/motion';
	import { linear as easing } from 'svelte/easing';
	import { fly } from 'svelte/transition';

	const dispatch = createEventDispatcher();

	export let countdown;

	let now = Date.now();
	let end = now + countdown * 1000;

	function updateTimer() {
    now = Date.now();
    console.log("count: ",count," / end: ",end," / countdown: ",countdown);
    if (countdown === -1)
    {
      handleFinishIt();
    }
    else
    {
      if (count === 0) // vérifie a chaque update si on est tombé à 0, si c'est le cas on déclenche un evenement timerend
      {
          async function waitOneSecond() {await new Promise(resolve => setTimeout(resolve, 1000));}
          //console.log("UPDATE BEFORE count: ",count," / end: ",end," / countdown: ",countdown);
          //dispatch('timerEnd'); // event timerEnd
          end = now + countdown * 1000;
          count = Math.round((end - now) / 1000);
          //console.log("UPDATE AFTER count: ",count," / end: ",end," / countdown: ",countdown);
          //alert("timer à 0");
          style_text_timer = true;
      }
      if (count === 1)
      {
          dispatch('timerEnd'); // on envoie l'event après, sinon on rencontre un probleme de priorité
      }
      if (count === 6)
      {
          alarm_id.volume = 0.1;
          alarm_id.play();
      }
    }
	}

	$: count = Math.round((end - now) / 1000);
	$: h = Math.floor(count / 3600);
	$: m = Math.floor((count - h * 3600) / 60);
	$: s = count - h * 3600 - m * 60;

	let interval = setInterval(updateTimer, 1000);
	$: if (count === -1) clearInterval(interval);

	let isResetting;
  let isPaused;
	const duration = 1000;

	let offset = tweened(1, { duration, easing });
	let rotation = tweened(360, { duration, easing });

	$: offset.set(Math.max(count - 1, 0) / countdown);
	$: rotation.set((Math.max(count - 1, 0) / countdown) * 360);

  // gérer l'affichage
  let alarm_id;
  let style_start = true;
  let style_skip = true;
  let style_pause = true;
  let style_mini_circle = true;
  let style_text_timer = false;
  let custom_text_function = true;
  let style_text_custom = "⠀Commencer!";
  let circleRadius = tweened(46);
  let pathColor = tweened({ h: 208, s: 100, l: 50 });

  // de base en pause
  handlePause();

  // Gestion de la création d'un interval / Start / Pause / Reset / Skip et Finish
	function handleNew() {
    clearInterval(interval);
    dispatch('new');
	}

	function handleStart() {
    now = Date.now();
    end = now + count * 1000;
    interval = setInterval(updateTimer, 1000);
    offset.set(Math.max(count - 1, 0) / countdown);
    rotation.set((Math.max(count - 1, 0) / countdown) * 360);
    isPaused = false;
    style_text_timer = true;
    custom_text_function = false;
	}

	function handlePause() {
    offset.set(count / countdown);
    rotation.set((count / countdown) * 360);
    clearInterval(interval);
    isPaused = true;
	}

	function handleReset() {
    clearInterval(interval);
    isResetting = true;
    isPaused = false;
    Promise.all([offset.set(1), rotation.set(360)]).then(() => {
    isResetting = false;
    now = Date.now();
    end = now + countdown * 1000;
    interval = setInterval(updateTimer, 1000);
    });
	}

	function handleSkip() {
    count=1;
		end=now+1*1000;
    style_text_custom = "⠀⋙";
    custom_text_function = false;
    style_text_timer = false;
	}

  function handleFinishIt() {
    style_start = false;
    style_skip = false;
    style_pause = false;
    style_mini_circle = false;
    circleRadius.set(40);
    pathColor.set({ h: 0, s: 100, l: 50 });
    style_text_timer = false;
    style_text_custom = " C'est prêt !";
  }
  /////////////////////////////////////////////////////////////

	function padValue(value, length = 2, char = '0') {
    const { length: currentLength } = value.toString();
    if (currentLength >= length) return value.toString();
    return `${char.repeat(length - currentLength)}${value}`;
	}

	onDestroy(() => {
	  clearInterval(interval);
	});
</script>
  
<main>
  <audio bind:this={alarm_id} src="https://dl.sndup.net/snd6/alarm.mp3"></audio>
  <svg in:fly={{ y: -5 }} viewBox="-50 -50 100 100" width="250" height="250">
    <title>Secondes restantes: {count}</title>
    <g fill="none" stroke="currentColor" stroke-width="2">
      <circle stroke="currentColor" r={$circleRadius} />
      <path
        stroke="hsl({$pathColor.h}, {$pathColor.s}%, {$pathColor.l}%)"
        d="M 0 -46 a 46 46 0 0 0 0 92 46 46 0 0 0 0 -92"
        pathLength="1"
        stroke-dasharray="1"
        stroke-dashoffset={$offset}
      />
    </g>
    <g fill="hsl(208, 100%, 50%)" stroke="none">
      <g transform="rotate({$rotation})">
        <g transform="translate(0 -46)">
          <circle r="4" style="display: {style_mini_circle ? 'block' : 'none'}" />
        </g>
      </g>
    </g>

    <g
      fill="currentColor"
      text-anchor="middle"
      dominant-baseline="baseline"
      font-size="13"
    >
      <text x="-3" y="6.5" style="visibility: {style_text_timer ? 'visible' : 'hidden'}">
        <a href={null} on:click={handlePause}>
        {#each Object.entries({ h, m, s }) as [key, value], i}
          {#if countdown >= 60 ** (2 - i)}
            <tspan dx="3" font-weight="bold">{padValue(value)}</tspan><tspan
              dx="0.5"
              font-size="7">{key}</tspan
            >
          {/if}
        {/each}
        </a>
      </text>
      <text x="-3" y="6.5" fill="#444" style="visibility: {style_text_timer ? 'hidden' : 'visible'}">
        <a href={null} on:click={custom_text_function ? handleStart : undefined}>
          {style_text_custom}
        </a>
      </text>
    </g>
  </svg>

  <div in:fly={{ y: -10, delay: 120 }}>
    <!-- <button on:click={handleNew}>New timer</button> -->
    <button on:click={handleReset} style="display: {style_skip ? 'block' : 'none'}">Redémarrer</button>

    {#if isPaused}
      <button disabled={isResetting || count === 0} on:click={handleStart} style="display: {style_pause ? 'block' : 'none'}">
        <span class="visually-hidden">Reprendre</span>

        <svg viewBox="-50 -50 100 100" width="30" height="30">
          <g
            fill="none"
            stroke="currentColor"
            stroke-width="20"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M -25 -40 l 60 40 -60 40z" />
          </g>
        </svg>
      </button>
    {:else}
      <button disabled={isResetting || count === 0} on:click={handlePause} style="display: {style_pause ? 'block' : 'none'}">
        <span class="visually-hidden">Pause</span>
        <svg viewBox="-50 -50 100 100" width="30" height="30">
          <g
            fill="none"
            stroke="currentColor"
            stroke-width="20"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M -25 -30 v 60 m 50 0 v -60" />
          </g>
        </svg>
      </button>
    {/if}

    <!-- <button on:click={handleReset}>Reset timer</button> -->
  <button on:click={handleSkip} style="display: {style_skip ? 'block' : 'none'}">Passer l'étape</button> 
  </div>
</main>

<style>
  /*  Main: */
  main > svg {
    width: 100%;
    height: auto;
    display: block;
    margin: 0 auto 2rem;
  }

  main > div {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 1rem;
    justify-content: center;
  }

  main > div button {
    font-size: 1rem;
    color: hsl(208, 100%, 50%);
    border: none;
    background: none;
    cursor: pointer;
    transition: color 0.2s ease-in-out;
  } 

  main > div button:hover {
    color: hsl(208, 100%, 40%);
  }

  main > div button:disabled {
    color: hsl(0, 0%, 100%);
    cursor: not-allowed;
  }


  @supports (display: grid) {
    div {
      display: grid;
      grid-template-columns: repeat(3, minmax(0, 1fr));
      justify-content: initial;
      justify-items: center;
    }
  }

  /* Boutons Redémarrer et Passer l'étape */
  button:nth-of-type(odd) {
    font-size: 0.9rem;
    border: none;
    text-transform: capitalize;
    padding: 0.5rem 1rem;
    background-color: hsl(208, 100%, 50%);
    color: white;
    border-radius: 0.25rem;
    cursor: pointer;
    transition: background-color 0.2s ease-in-out;
  }

  /* Hover Boutons Redémarrer et Passer l'étape */
  button:nth-of-type(odd):hover {
    background-color: hsl(0, 0%, 100%);
  }

  button:nth-of-type(odd):focus {
    outline: none;
    box-shadow: 0 0 0 2px rgb(255, 255, 255), 0 0 0 4px hsl(208, 100%, 50%);
  }

  button:nth-of-type(2) {
    color: inherit;
    width: 3rem;
    height: 3rem;
    border-radius: 50%;
    border: none;
    padding: 1rem;
    background: hsl(208, 100%, 50%);
    box-shadow: 0px 1px 2px hsl(208, 100%, 50%);
    transition: box-shadow 0.2s ease-in-out, transform 0.25s ease-in-out;
  }

  button:nth-of-type(2):hover {
    color: inherit;
    background: hsl(0, 0%, 100%);
    box-shadow: 0px 1px 2px hsl(0, 0%, 100%);

  }

  button:nth-of-type(2) svg {
    margin: initial;
    width: 100%;
    height: auto;
    display: block;
  }

  button:nth-of-type(2):hover svg {
    color: hsl(208, 100%, 50%);
  }

  button[disabled] {
    transform: scale(0);
  }

  :global(.timer_wrapper) {
    width: 50%;
    color: #ffffff;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen-Sans, Ubuntu, Cantarell, 'Helvetica Neue', sans-serif; 
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    margin: 1em;
  }


  :global(.visually-hidden:not(:focus):not(:active)) {
    clip: rect(0 0 0 0);
    clip-path: inset(50%);
    height: 1px;
    overflow: hidden;
    position: absolute;
    white-space: nowrap;
    width: 1px;
  }
</style>