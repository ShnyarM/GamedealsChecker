<script>
  // @ts-ignore
  import axios from 'axios';
  // @ts-ignore
  import {onMount} from 'svelte';
  // @ts-ignore
  import GameElement from "/src/GameElement.svelte";
  // @ts-ignore
  import DealView from "/src/DealView.svelte";

  let gameName = "" //search input value
  let minPrice = ""
  let maxPrice = ""
  let onlyOnSale = false //on sale checkbox value
  let selectedStores = ["Steam", "GameStop", "GOG", "Origin", "Uplay", "Epic Games Store", "Blizzard Shop"] //Stores which to show, bound to checkbox list
  const sortSelection = ["Deal Rating", "Title", "Savings", "Price", "Metacritic", "Reviews", "Release", "Store"] //Possible sort posibilities
  let sortSelected = "Deal Rating" //Selected sort

  let currentPage = 0

  let showStores=false //Show store checkboxes
  let showSort=false //show sort selection

  let storeInfo = [] //information about different stores
  let gameSearchPromise //promise of api call to get games
  let dealSearchPromise //promise of api call to get deal
  let gameList = [] //list of deals from api
  let dealDetails = {} //information about specific deal
  let viewDeal = "" //current deal to look at, empty if searching for games

  onMount(async () => {
    getStoreInfo()//get information about stores
  });

  //get information about stores
  async function getStoreInfo(){
    const options = {
      method: 'GET',
      url: 'https://cheapshark-game-deals.p.rapidapi.com/stores',
      headers: {
      'X-RapidAPI-Key': '13a978fec3mshe18fa469f4e5b4ep1c0eb2jsn227fdc7214be',
      'X-RapidAPI-Host': 'cheapshark-game-deals.p.rapidapi.com'
      }
    };
    const response = await axios.request(options)
    storeInfo = response.data
    console.log(storeInfo.length)
  }

  //Search for games
  async function searchGame(){
    //Convert min and max price to usable numbers
    if(isNaN(parseFloat(maxPrice))) maxPrice=""
    else if(parseFloat(maxPrice) < 0) maxPrice= (-parseFloat(maxPrice)).toString()

    if(isNaN(parseFloat(minPrice))) minPrice=""
    else if(parseFloat(minPrice) < 0) minPrice= (-parseFloat(minPrice)).toString()

    //Make array of stores which to include
    const stores = []
    for(let i = 0; i < storeInfo.length; i++) if(selectedStores.indexOf(storeInfo[i].storeName) != -1) stores.push(i+1)

    const options = {
      method: 'GET',
      url: 'https://cheapshark-game-deals.p.rapidapi.com/deals',
      params: {
        storeID: stores,
        onSale: onlyOnSale ? "1" : "0",
        pageNumber: currentPage,
        sortBy: sortSelected,
        title: gameName,
        upperPrice: maxPrice == "" ? "50" : maxPrice,
        lowerPrice: minPrice
      },
      headers: {
        'X-RapidAPI-Key': '13a978fec3mshe18fa469f4e5b4ep1c0eb2jsn227fdc7214be',
        'X-RapidAPI-Host': 'cheapshark-game-deals.p.rapidapi.com'
      }
    };

    const response = await axios.request(options)

    //Check if games were found
    if(response.data[0])  {gameList = response.data; return true}
    else throw new Error("Nothing was found");
  }

  //Get details about specific deal
  async function getDeal(){
    const options = {
      method: 'GET',
      url: 'https://cheapshark-game-deals.p.rapidapi.com/deals',
      params: {
        id:decodeURIComponent(viewDeal) //decode string first before sending it, else it would be double-encoded
      },
      headers: {
        'X-RapidAPI-Key': '13a978fec3mshe18fa469f4e5b4ep1c0eb2jsn227fdc7214be',
        'X-RapidAPI-Host': 'cheapshark-game-deals.p.rapidapi.com'
      }
    };

    const response = await axios.request(options)

    //Check if games were found
    if(response.data.length != 0)  {dealDetails = response.data; return true}
    else throw new Error("Something went wrong");
  }

  //Handle message from gameElement component
  function handleMessage(event){
    viewDeal = event.detail.dealID
    dealSearchPromise = getDeal()
  }

  //Select all stores
  function selectAllStores(){
    selectedStores = []
    for(let i = 0; i < storeInfo.length; i++) selectedStores.push(storeInfo[i].storeName)
  }

  //detect if enter key was hit to activate search
  document.addEventListener("keydown", e => {
    if (e.key === "Enter") {
      currentPage = 0; 
      gameSearchPromise = searchGame(); 
      showSort = false; 
      showStores = false
    }
  });
</script>

<main>
  <!--Search Page-->
  {#if viewDeal==""}
    <h1><img src="/controller.png" alt="controller" style="height:100px; width:auto; vertical-align:middle">Gamedeals Checker</h1>
    <h2>Search Options</h2>
    
    <!--Top Row of search options-->
    <div class="inputDiv" style="margin-bottom: 10px">
      <input type="text" class="textInput" placeholder="Game Title" bind:value={gameName}>

      <input type="number" class="textInput" placeholder="Minimum Price ($)" bind:value={minPrice}>

      <input type="number" class="textInput" placeholder="Maximum Price ($)" bind:value={maxPrice}>
    </div>

    <!--Second Row of search options-->
    <div class="inputDiv" style="align-items: {(showStores || showSort) ? "flex-start" : "center"};">
      <!--Store selection-->
      <div>
        <button on:click={() => showStores = !showStores} style="width:181px">Show Selected Stores</button>
        {#if showStores}
          <br><br>
          <button style="height: 30px" on:click={selectAllStores}>Select All</button><br>
          <button style="height: 30px; margin-top:5px; margin-bottom:5px;" on:click={() => selectedStores = []}>Deselect All</button>
          <br>
          {#each storeInfo as store}
            <label>
              <input type=checkbox bind:group={selectedStores} name="stores" value={store.storeName}>
              {store.storeName}<br>
            </label>
          {/each}
        {/if}
      </div>

      <!--Sort option selection-->
      <div>
        <button on:click={() => showSort = !showSort} style="width:181px">Show Sort Options</button>
        {#if showSort}
          <br><br>
          {#each sortSelection as sort}
            <label>
              <input type=radio bind:group={sortSelected} name="sort" value={sort}>
              {sort}<br>
            </label>
          {/each}
        {/if}
      </div>

      <div style="width:181px">
        <input type="checkbox" name="checkbox" bind:checked={onlyOnSale}>
        <label for="checkbox"> On Sale</label><br>
      </div>
    </div>

    <button id="searchButton" on:click={() => {currentPage = 0; gameSearchPromise = searchGame(); showSort = false; showStores = false}}>Search</button>

    <!--list of games-->
    <div>
      {#await gameSearchPromise}
        <h2>Loading...</h2>
      {:then}
        {#each gameList as game, i}
          <GameElement game={game} left={i%2 == 0 ? 1 : 0} storeInfo={storeInfo} on:message={handleMessage}/>
        {/each}
        <!--page indicator and buttons-->
        {#if gameList.length != 0}
          <div style="display:flex; justify-content:space-evenly; margin:20px">
            <div style="display:flex; align-items: center;">
              {#if currentPage != 0}
                <button style="margin-right: 10px; height:30px;" on:click={() => {currentPage--; gameSearchPromise=searchGame()}}>{"<"}</button>
              {/if}
              <p style="margin:0px">Page {currentPage+1}</p>
              {#if gameList.length == 60}
                <button style="margin-left: 10px; height:30px;" on:click={() => {currentPage++; gameSearchPromise=searchGame()}}>{">"}</button>
              {/if}
            </div>
          </div>
        {/if}
      <!--error message-->
      {:catch error}
        <h2 style="color: red">{error.message}</h2>
      {/await}
    </div>


  <!--Deal View page-->
  {:else}
    <button style="position:absolute; top:10px; left:10px" on:click={() => viewDeal=""} on:keydown={() => viewDeal=""}>Back</button>
    {#await dealSearchPromise}
      <h1>Loading...</h1>
    {:then}
      <DealView dealDetails={dealDetails} viewDeal={viewDeal} storeInfo={storeInfo} on:message={handleMessage}/>
    {:catch error}
      <h2 style="color: red">{error.message}</h2>
    {/await}
  {/if}
</main>
