<!-- <h1>Welcome to SvelteKit</h1> -->
<!-- <p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p> -->

<script>
    let maxClick = 4;
    let cnt = maxClick; // tip: https://svelte.dev/docs/svelte/$state
    
  
    function onClick() {
      // tip: Since DOM (i.e., the webpage content) will automatically update based on values, [<p id="info">Remaining Number of Clicks: {cnt}</p>]
      // we only need to change the cnt number here.
        if (cnt > 0) {
            cnt -= 1;
            if (cnt === 0) {
                alert("No more clicks remaining!");
            }
        }
    }

  </script>
  
  <h1>Evan Lembke's VIS Site</h1>
  <img
    width="200px"
    src="https://play-lh.googleusercontent.com/Q1C4YGtT8fnx4CK4T75sxr5sz-Qz_DChF8Gd3Az_laMI_Uw2iOXC3RzsMcjioqpTFGE=w240-h480-rw"
  />
  <div>
    You can click up to
    <select 
        bind:value={maxClick/*tip: bind the select action to change the maxClick value. https://svelte.dev/docs/svelte/bind#select-bind:value */} 
        onchange={() => {cnt = Number(maxClick);}/*tip: define what will happen after click. Maybe you want to update the remaining number of clikc when click a new maxClick value */}>
        {#each [2, 4, 6] as optionNum}
            <option value={optionNum}>
                {optionNum}
            </option>
        {/each}
    </select>
    times
  </div>
  <button onclick={onClick}> Click Me </button>
    

    {#if cnt > 0}
        <p id="info">Remaining Number of Clicks: {cnt}</p>
    {:else}
        <p>No more clicks allowed</p>
    {/if}

    <!-- tip: use {#if...} template syntax here (https://svelte.dev/docs/svelte/if) so that the content below will automatically update when cnt value changes -->
    <!-- `the content below should only show when cnt >0` -->
    <!-- <p id="info">Remaining Number of Clicks: {cnt}</p> -->
    <!-- The content below should only shown when cnt =0 -->
    <!-- <p>No more clicks allowed</p> -->
  
  
  <style>
    body {
      font-family: Arial, Helvetica, sans-serif;
    }
    button {
      background-color: #44aa66;
      /* background-color: blue; */
      color: white;
      font-size: xx-large;
      padding: 10px 20px;
      border: none;
      cursor: pointer;
      border-radius: 5px;
    }
  </style>