<script>
  import { createEventDispatcher } from "svelte";
  import { scale } from "svelte/transition";
  import { elasticOut } from "svelte/easing";
  export let celeb;
  export let showPrice;
  export let winner;
  const dispatch = createEventDispatcher();
</script>

<style>
  .card-outer {
    width: 100%;
    height: 100%;
  }
  .card-inner {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background: 50% 0 no-repeat;
    background-size: cover;
    overflow: hidden;
    padding: 0;
    text-align: left;
    border-radius: var(--border-radius);
    box-shadow: 2px 4px 6px rgba(0, 0, 0, 0.2);
  }

  .details {
    position: absolute;
    width: 100%;
    bottom: 0;
    padding: 1em 0.5em 0.5em 0.5em;
    background: linear-gradient(to bottom, transparent, black);
    color: white;
  }

  h2 {
    margin: 0 0 0.2em 0;
  }

  .type {
    margin: 0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  @media (min-width: 640px) {
    .card-outer {
      height: 0;
      padding: 0 0 100% 0;
    }
  }

  .price {
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    background-color: rgba(0, 0, 0, 0.5);
    color: white;
    display: flex;
    align-items: center;
  }

  .price.large {
    font-size: 6em;
  }
</style>

<div class="card-outer">
  <button
    class="card-inner"
    style="background-image: url({celeb.image})"
    on:click={() => dispatch('select')}>
    <div class="details">
      <h2>
        <a target="_blank" href="something">{celeb.name}</a>
      </h2>
      <p class="type">{celeb.type}</p>
    </div>
    {#if showPrice}
      <div class="price" class:large={winner}>
        <span in:scale={{ easing: elasticOut, duration: 200 }}>
          ${celeb.price}
        </span>
      </div>
    {/if}
  </button>
</div>
