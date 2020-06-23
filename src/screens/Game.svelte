<script>
  import { createEventDispatcher } from "svelte";
  import { fly, crossfade, scale } from "svelte/transition";
  import * as eases from "svelte/easing";
  import Card from "../components/Card.svelte";
  import { sleep, pick_random, loadImages } from "../utils";
  export let selection;
  const dispatch = createEventDispatcher();
  const loadDetails = async (celeb) => {
    const res = await fetch(
      `https://cameo-explorer.netlify.app/celebs/${celeb.id}.json`
    );
    const details = await res.json();
    await loadImages(details.image);
    return details;
  };
  const promises = selection.map((round) =>
    Promise.all([loadDetails(round.a), loadDetails(round.b)])
  );
  const [send, receive] = crossfade({
    easing: eases.cubicInOut,
    duration: 300,
  });

  const results = Array.from({ length: selection.length });

  let i = 0;
  let lastResult;
  let done = false;
  let ready = true;
  $: score = results.filter((x) => x === "right").length;
  const pickMsg = (p) => {
    if (p < 0.5) return pick_random(["Ouch", "Could be better"]);
    else if (p <= 0.8) return pick_random(["Not Bad", "Keep Practicing"]);
    else if (p <= 0.9) return pick_random(["Cool", "Close to perfect"]);
    else return pick_random(["Great!", "Awesome!"]);
  };

  const submit = async (a, b, sign) => {
    lastResult = Math.sign(a.price - b.price) === sign ? "right" : "wrong";
    await sleep(1500);
    results[i] = lastResult;
    lastResult = null;
    await sleep(500);
    if (i < selection.length - 1) {
      i++;
    } else {
      done = true;
    }
  };
</script>

<style>
  .game-container {
    flex: 1;
  }

  .game {
    display: grid;
    grid-template-rows: 1fr 2em 1fr;
    grid-gap: 0.5em;
    width: 100%;
    height: 100%;
    max-width: min(100% 40vh);
    margin: 0 auto;
  }

  .giant-result {
    position: fixed;
    width: 50vmin;
    height: 50vmin;
    left: calc(50vw - 25vmin);
    top: calc(50vw - 25vmin);
    opacity: 0.5;
  }

  .game > div {
    display: flex;
    align-items: center;
  }

  @media (min-width: 640px) {
    .game {
      max-width: 100%;
      grid-template-rows: none;
      grid-template-columns: 1fr 8em 1fr;
    }
    .same {
      height: 8em;
    }
  }

  .results {
    display: grid;
    grid-gap: 0.2em;
    width: 100%;
    max-width: 320px;
    margin: 1em auto 0 auto;
  }

  .result {
    background-color: gray;
    border-radius: 50%;
    padding: 0 0 100% 0;
    transition: background 0.2s;
    transition-delay: 0.2s;
  }
  .result img {
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
  }

  .done {
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .done strong {
    font-size: 6em;
    font-weight: 700;
  }
</style>

<header>
  <p>
    Tap on the more monetisable celebrity's face, or tap 'same price' if society
    values them equally.
  </p>
</header>
<div class="game-container">
  {#if done}
    <div
      class="done"
      in:scale={{ delay: 200, duration: 800, easing: eases.elasticOut }}>
      <strong>{score} / {results.length}</strong>
      <p>{pickMsg(score / results.length)}</p>
      <button on:click={() => dispatch('restart')}>back to home</button>

    </div>
  {:else if ready}
    {#await promises[i] then [a, b]}
      <div
        class="game"
        in:fly={{ duration: 200, y: 20 }}
        out:fly={{ duration: 200, y: -20 }}
        on:outrostart={() => (ready = false)}
        on:outroend={() => (ready = true)}>
        <div class="card-container">
          <Card
            celeb={a}
            on:select={() => submit(a, b, 1)}
            showPrice={!!lastResult}
            winner={a.price > b.price} />
        </div>

        <div>
          <button class="same" on:click={() => submit(a, b, 0)}>
            same price
          </button>
        </div>

        <div class="card-container">
          <Card
            celeb={b}
            on:select={() => submit(a, b, -1)}
            showPrice={!!lastResult}
            winner={b.price > a.price} />
        </div>
      </div>
    {:catch}
      <p>something wrong</p>
    {/await}
  {/if}

</div>
{#if lastResult}
  <img
    in:fly={{ x: 100, duration: 200 }}
    out:send={{ key: i }}
    class="giant-result"
    alt={lastResult}
    src="/icons/{lastResult}.svg" />
{/if}

<div
  class="results"
  style="grid-template-columns: repeat({results.length}, 1fr)">
  {#each results as result, i}
    <span class="result">
      {#if result}
        <img in:receive={{ key: i }} alt={result} src="/icons/{result}.svg" />
      {/if}
    </span>
  {/each}
</div>
