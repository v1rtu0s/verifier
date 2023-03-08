<style>
  @import "https://unpkg.com/chota@latest";

  a {
    margin: 4px;
  }

  nav {
    margin-bottom: 1rem;
  }

  .active {
    color: #ff3d03;
    font-weight: 700;
  }

  .button {
    margin: 1rem;
  }

  label {
    display: inline-block;
    width: 210px;
    text-align: center;
  }
</style>

<script>
  import { Button, Input, Field, Card, Container } from "svelte-chota";
  import { createHmac } from "crypto";
  import { onMount } from "svelte";
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";

  let serverSeed = "";
  let clientSeed = "";
  let nonce = "";
  let cursor = 0;
  let count = 20;
  let results = [];
  let display_bytes = [];
  let DisplayResult = [];
  let verified_bytes = [];
  let verified_floats = [];

  const generateRandomNumbers = () => {
    results = generateFloats({ serverSeed, clientSeed, nonce, cursor, count });
  };
  const generateDisplayBytes = () => {
    display_bytes = generateBytes({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count
    });
  };
  const generateDisplayResult = () => {
    const DisplayResult = generateResult({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count
    });
  };

  function verify_floats() {
    verified_floats = generateResult({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count
    });
    return verified_floats;
  }

  function verify_bytes() {
    verified_bytes = generateBytes({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count
    });
    return verified_bytes;
  }

  $: verified_floats = generateFloats({
    serverSeed,
    clientSeed,
    nonce,
    cursor,
    count
  });

  $: verified_bytes = generateBytes({
    serverSeed,
    clientSeed,
    nonce,
    cursor,
    count
  });

  function generateResult({ serverSeed, clientSeed, nonce, cursor, count }) {
    ////////////// LIMBO
    if (activeTab === "limbo") {
      let endresult =
        generateFloats({ serverSeed, clientSeed, nonce, cursor, count }) *
        16777216;
      endresult = (16777216 / (endresult + 1)) * (1 - 0.01);
      return endresult.toFixed(2);
    }

    ////////////// DICE
    if (activeTab === "dice") {
      let endresult = 0;
      let endresult2 =
        generateFloats({ serverSeed, clientSeed, nonce, cursor, count }) * 10001;
      endresult2 = Math.trunc(endresult2);
      endresult = endresult2 / 100;
      return endresult;
    }

    ////////////// DIAMONDS
    if (activeTab === "diamonds") {
      count = 20;
      let endresult = generateFloats({
        serverSeed,
        clientSeed,
        nonce,
        cursor,
        count
      });
      let bytes = generateBytes({ serverSeed, clientSeed, nonce, cursor, count });

      console.log(bytes);
      console.log(endresult);

      return endresult;
    }
  }

  function generateBytes({ serverSeed, clientSeed, nonce, cursor, count }) {
    // Setup cursor variables
    let currentRound = Math.floor(cursor / 32);
    let currentRoundCursor = cursor;
    currentRoundCursor -= currentRound * 32;

    // Generate bytes until count is fulfilled
    const bytes = [];
    while (bytes.length < count) {
      // HMAC function used to output provided inputs into bytes
      const hmac = createHmac("sha256", serverSeed);
      hmac.update(`${clientSeed}:${nonce}:${currentRound}`);
      const buffer = hmac.digest();

      // Update cursor for next iteration of loop
      while (currentRoundCursor < 32 && bytes.length < count) {
        bytes.push(buffer[currentRoundCursor]);
        currentRoundCursor += 1;
      }
      currentRoundCursor = 0;
      currentRound += 1;
    }
    console.log(bytes);
    return bytes;
  }

  function generateFloats({ serverSeed, clientSeed, nonce, cursor, count }) {
    const bytes = generateBytes({
      serverSeed,
      clientSeed,
      nonce,
      cursor,
      count: count * 4
    });

    // Convert bytes to floats
    const floats = [];
    for (let i = 0; i < bytes.length; i += 4) {
      let partial = 0;
      for (let j = 0; j < 4; j++) {
        partial += bytes[i + j] / 256 ** (j + 1);
      }
      floats.push(partial);
    }
    console.log(floats);
    return floats;
  }

  let activeTab = "limbo";

  const animatedNumber = tweened(0, {
    duration: 400,
    easing: cubicOut
  });

  $: generatedResult = parseFloat(
    generateResult({ serverSeed, clientSeed, nonce, cursor, count })
  ).toFixed(2);
  function animateNumber() {
    animatedNumber.set(parseFloat(generatedResult));
  }
</script>

<nav>
  
<a class:active={activeTab === 'limbo'} href="#" on:click={() => activeTab = 'limbo'}>Limbo</a>
  <a class:active={activeTab === 'dice'} href="#" on:click={() => activeTab = 'dice'}>Dice</a>
  <a class:active={activeTab === 'diamonds'} href="#" on:click={() => activeTab = 'diamonds'}>Diamonds</a>
  <a class:active={activeTab === 'wheel'} href="#" on:click={() => activeTab = 'wheel'}>Wheel</a>  
</nav>

<Container>
<label>
  Server Seed:
  <input bind:value={serverSeed} />
</label>

<br>

<label>
  Client Seed:
  <input bind:value={clientSeed} />
</label>

<br>


<label>
  Nonce:
  <input bind:value={nonce} />
</label>
<br>

<div style="margin-left:1rem">

<a class="button primary" on:click={verify_bytes}>VERIFY</a>
<a class="button primary" on:click={animateNumber}>test</a>
</div>


{#each {verified_bytes} as test}
{test}
{/each}


<!-- <table>
		<thead>
			<tr>
				<th>{clientSeed}{verified_bytes}</th>

			</tr>
		</thead>
		<tbody>
  {#each verified_bytes() as row}
    <tr>
      {#each verified_bytes(row) as cell}
        <td>{verified_bytes}</td>
      {/each}
    </tr>
  {/each}
</tbody>

	</table> -->





<!-- 
{#if display_bytes.length > 0 && clientSeed.length > 0 && serverSeed.length > 0 && nonce.length > 0}
 <ul>
ENDRESULT: {generateResult({ serverSeed, clientSeed, nonce, cursor, count })}
<br>
{$animatedNumber}
{generatedResult}

</ul>
________________
<br>

breakdown
  <ul>
    
      FLOATS: {generateFloats({ serverSeed, clientSeed, nonce, cursor, count })}
    
  </ul>
{/if}
{#if display_bytes.length > 0 && clientSeed.length > 0 && serverSeed.length > 0 && nonce.length > 0}
<ul>
BYTES: {display_bytes} {generateBytes({ serverSeed, clientSeed, nonce, cursor, count })}

</ul>
<ul>
{DisplayResult} 
</ul>

{/if} -->
</Container>

