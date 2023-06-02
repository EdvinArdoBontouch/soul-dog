<script>
  import "../app.css";
  import { onMount } from "svelte";
  import { DoubleBounce } from "svelte-loading-spinners";

  let dogUrl = null;
  let breeds = [];
  let name = "";
  let fetched = false;
  let loading = false;

  const getHashFromString = (str) => {
    let hash = 0,
      i,
      chr;
    if (str.length === 0) return hash;
    for (i = 0; i < str.length; i++) {
      chr = str.charCodeAt(i);
      hash = (hash << 5) - hash + chr;
      hash |= 0; // Convert to 32bit integer
    }
    return hash;
  };

  const fetchImage = async () => {
    const response = await fetch("https://dog.ceo/api/breeds/image/random");
    return await response.json();
  };

  const fetchBreeds = async () => {
    const response = await fetch("https://dog.ceo/api/breeds/list/all");
    breeds = (await response.json()).message;
  };

  const fetchByBreed = async () => {
    const breed = getBreedFromName();
    const response = await fetch(`https://dog.ceo/api/breed/${breed}/images`);
    const variations = (await response.json()).message;
    const hash = Math.abs(getHashFromString(name.toLowerCase()));
    const variationIndex = hash % variations.length;

    return variations[variationIndex];
  };

  const getBreedFromName = () => {
    const breedNames = Object.keys(breeds);
    const hash = Math.abs(getHashFromString(name.toLowerCase()));
    const breedIndex = hash % breedNames.length;
    return breedNames[breedIndex];
  };

  onMount(async () => {
    const response = await fetchImage();
    dogUrl = response.message;
    await fetchBreeds();
  });

  const getSoulDog = async () => {
    loading = true;
    dogUrl = await fetchByBreed();
    fetched = true;
    loading = false;
    console.log("dogUrl", dogUrl);
    console.log(name, getHashFromString(name));
  };
</script>

<main class="flex items-center justify-between flex-col mt-5">
  <h1 class="text-8xl font-bold animate-character mb-8">Soul Dog 🐩</h1>
  <div class="w-80 p-4 bg-white shadow rounded">
    <form on:submit|preventDefault={getSoulDog} class="flex flex-col">
      <input
        type="text"
        class="bg-gray-50 border border-gray-300 rounded-md py-2 px-3 mb-3 focus:outline-orange-600"
        bind:value={name}
      />
      <button
        class="py-2 px-3 text-white text-sm font-semibold rounded-md shadow focus:outline-none bg-gradient-to-r from-orange-600 to-orange-400"
        on:click={getSoulDog}
      >
        Get your soul dog
      </button>
    </form>

    {#if loading}
      <div class="my-3 h-52 w-full flex items-center justify-center">
        <DoubleBounce color="#FF3E00" />
      </div>
    {/if}

    {#if fetched && !loading}
      <img src={dogUrl} alt="" class="my-3 w-full" />
      <h2 class="text-xl text-center text-indigo-600">
        Here you are, {name}! ✨
      </h2>
    {/if}
  </div>
</main>

<style>
  .animate-character {
    text-transform: uppercase;
    background-image: linear-gradient(
      -225deg,
      #231557 0%,
      #44107a 29%,
      #ff1361 67%,
      #fff800 100%
    );
    background-size: auto auto;
    background-clip: border-box;
    background-size: 200% auto;
    color: #fff;
    background-clip: text;
    text-fill-color: transparent;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    animation: textclip 2s linear infinite;
    display: inline-block;
  }

  @keyframes textclip {
    to {
      background-position: 200% center;
    }
  }
</style>
