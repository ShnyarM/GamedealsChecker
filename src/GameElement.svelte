<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  export let game; //information about game
  export let left; //says if element is on the left or the right, needed for margins
  export let storeInfo; //information about stores

  //Tell main app to open a deal
  function openDeal(dealID){
    dispatch("message", {dealID:dealID});
  }
</script>


<div class={left ? "gameDiv left" : "gameDiv right"} on:click={() => openDeal(game.dealID)} on:keydown={() => openDeal(game.dealID)}>
  <p class="gameTitle"><strong>{game.title}</strong></p>

  <div style="display:flex;">
    <img src={game.thumb} alt="Game Cover" class="gameImg">

    <div style="margin-left: 4px;">
      <p class="infoText">Current Price</p>
      <p class="infoText">{game.salePrice + "$"}</p>
    </div>

    <div>
      <p class="infoText">Normal Price</p>
      <p class="infoText">{game.normalPrice + "$"}</p>
    </div>

    <div>
      <p class="infoText">On Sale?</p>
      {#if game.isOnSale == 0}
        <img src="src/images/red_cross.png" alt="red-cross" style="width:20px; height:20px">
      {:else}
        <p class="infoText" style="color:greenyellow">{parseInt(game.savings) + "%"}</p>
      {/if}
    </div>
  </div>

  <div style="display:flex; justify-content:space-between; margin-top:5px; margin-left: 5px;">

    <div style="display:flex; justify-content:space-between">
      <div style="margin-left: 4px;">
        <p class="infoText">Steam Rating</p>
        <p class="infoText" style="color: rgb({(100-game.steamRatingPercent)*2.55}, {game.steamRatingPercent*2.55}, 0)">{game.steamRatingPercent != 0 ? game.steamRatingPercent + "%" : "Not available"}</p>
      </div>
    </div>
    
    <img src={"https://www.cheapshark.com/img/stores/logos/"+(game.storeID-1)+".png"} alt={storeInfo[game.storeID-1].storeName} title={storeInfo[game.storeID-1].storeName} class="storeImg">
  </div>
</div>