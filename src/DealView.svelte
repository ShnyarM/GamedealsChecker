<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  export let dealDetails; //information about game
  export let storeInfo; //information about stores
  export let viewDeal;

  const gameInfo = dealDetails.gameInfo
  const onSale = gameInfo.retailPrice == gameInfo.salePrice ? false : true //Says if game is on sale
  const salePercentage = parseInt(((1-(parseFloat(gameInfo.salePrice)/parseFloat(gameInfo.retailPrice)))*100).toString()) //long calculation because type conversion
  const redirectLink = "https://www.cheapshark.com/redirect?dealID=" //Redirect link of CheapShark API

  //Function to convert unixtime in seconds into a date
  function unixTimeToDate(time){
    if(time == 0) return "Not available"

    const milliseconds = time*1000
    const date = new Date(milliseconds)

    const month = date.toLocaleString("en-US", {month: "long"})
    const day = date.toLocaleString("en-US", {day: "numeric"})
    const year = date.toLocaleString("en-US", {year: "numeric"})

    return month + " " + day + ", " + year
  }

  //Tell main app to view new Deal
  function openDeal(dealID){
    dispatch("message", {dealID:dealID});
  }
</script>


<h1 style="margin-bottom: 0px;">{gameInfo.name}</h1>
<h2>Store: {storeInfo[gameInfo.storeID-1].storeName}</h2>
<div style="display:flex; justify-content:space-between; margin-left:7%; margin-right: 7%;">

  <!--left side with information-->
  <div style="margin-bottom: 50px">
    <p class="dealText">Normal Price: {gameInfo.retailPrice + "$"}</p>
    <p class="dealText">Current Price: {gameInfo.salePrice + "$"}</p>

    <!--Show if on sale or not-->
    <p class="dealText" style="displeay:flex; color:{onSale ? "greenyellow" : "white"}">Sale: 
      {#if !onSale}
        <img src="src/images/red_cross.png" alt="red-cross" style="width:30px; height:30px; vertical-align:center">
      {:else}
        {salePercentage + "%"} <!--long calculation because type conversion-->
      {/if}
    </p>

    <p class="dealText">Release Date: {unixTimeToDate(gameInfo.releaseDate)}</p>

    <!--Steam Rating, very long because adjusting colors-->
    <p class="dealText" style="color: rgb({(100-gameInfo.steamRatingPercent)*2.55}, {gameInfo.steamRatingPercent*2.55}, 0)">
      Steam Rating: {gameInfo.steamRatingPercent != 0 ? gameInfo.steamRatingPercent + "%" : "Not available"}
    </p>

    <!--metacritic score, very long because adjusting colors and clickable link-->
    <p class="dealText clickableText" style="color: rgb({(100-gameInfo.metacriticScore)*2.55}, {gameInfo.metacriticScore*2.55}, 0)" 
      on:click={() => window.open("https://www.metacritic.com"+gameInfo.metacriticLink)} on:keydown={() => window.open("https://www.metacritic.com"+gameInfo.metacriticLink)}>
      Metacritic Score: {gameInfo.metacriticScore != 0 ? gameInfo.metacriticScore : "Not available"}
    </p>
     
    <!--Display all cheaper stores with clickable links-->
    <p class="dealText">Cheaper Stores: {dealDetails.cheaperStores.length}
      {#if dealDetails.cheaperStores.length != 0}
        {#each dealDetails.cheaperStores as store}
        <li style="margin-left:80px"><p class="dealText clickableText" on:click={() => openDeal(store.dealID)} on:keydown={() => openDeal(store.dealID)}>
          {storeInfo[store.storeID-1].storeName + ": " + store.salePrice + "$"}
        </p></li>
        {/each}
      {/if}
    </p>

    {#if dealDetails.cheapestPrice.date != 0}
      <p class="dealText">Cheapest Recorded Price: {dealDetails.cheapestPrice.price + "$, " + unixTimeToDate(dealDetails.cheapestPrice.date)}</p>
    {/if}
  </div>
        
  <!--right side with image and link-->
  <div>
    <img src={gameInfo.thumb} alt="Game Cover" style="text-align: right; width: 300px; height: auto;"><br>
    <a href={redirectLink + viewDeal} target="_blank" rel="noreferrer"><button style="width:300px">Go To Deal Page</button></a>
  </div>
</div>