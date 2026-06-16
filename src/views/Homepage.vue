<script setup >
// import ding from "https://unpkg.com/material-components-web@latest/dist/material-components-web.min.js"
import router from '../router/index.js';
import { ref, computed, watch, onMounted } from 'vue';
// const MDCBanner = mdc.banner.MDCBanner;const MDCCheckbox = mdc.checkbox.MDCCheckbox;const MDCChip = mdc.chips.MDCChip;const MDCChipSet = mdc.chips.MDCChipSet;const MDCCircularProgress = mdc.circularProgress.MDCCircularProgress;const MDCDataTable = mdc.dataTable.MDCDataTable;const MDCDialog = mdc.dialog.MDCDialog;const MDCDrawer = mdc.drawer.MDCDrawer;const MDCFloatingLabel = mdc.floatingLabel.MDCFloatingLabel;const MDCFormField = mdc.formField.MDCFormField;const MDCIconButtonToggle = mdc.iconButton.MDCIconButtonToggle;const MDCLineRipple = mdc.lineRipple.MDCLineRipple;const MDCLinearProgress = mdc.linearProgress.MDCLinearProgress;const MDCList = mdc.list.MDCList;const MDCMenu = mdc.menu.MDCMenu;const MDCMenuSurface = mdc.menuSurface.MDCMenuSurface;const MDCNotchedOutline = mdc.notchedOutline.MDCNotchedOutline;const MDCRadio = mdc.radio.MDCRadio;const MDCRipple = mdc.ripple.MDCRipple;const MDCSegmentedButton = mdc.segmentedButton.MDCSegmentedButton;const MDCSelect = mdc.select.MDCSelect;const MDCSlider = mdc.slider.MDCSlider;const MDCSnackbar = mdc.snackbar.MDCSnackbar;const MDCSwitch = mdc.switchControl.MDCSwitch;const MDCTabBar = mdc.tabBar.MDCTabBar;const MDCTextField = mdc.textField.MDCTextField;const MDCTooltip = mdc.tooltip.MDCTooltip;const MDCTopAppBar = mdc.topAppBar.MDCTopAppBar;

window.addEventListener('beforeinstallprompt', (e) => {
  // Niet tonen als de app al geïnstalleerd/open staat
  if (window.matchMedia('(display-mode: standalone)').matches) {
    return;
  }

  e.preventDefault();
  deferredPrompt = e;

  const installBanner = document.getElementById('installBanner');

  if (installBanner) {
    installBanner.style.display = 'flex';
  }
});

onMounted(() => {
  const isStandalone =
    window.matchMedia('(display-mode: standalone)').matches;

  if (isStandalone) {
    const installBanner = document.getElementById('installBanner');

    if (installBanner) {
      installBanner.style.display = 'none';
    }
  }
});
const pokemons = ref([]);
const pokemonIds = ref(0);
const loading = ref(true);
const error = ref(null);
const visibleCount = ref(20);
const showFavoritePokemons = ref(false);
const searchQuery = ref('');

const filteredPokemons = computed(() => {
    const source = showFavoritePokemons.value
        ? pokemons.value.filter(pokemon =>
              favoritePokemons.value.some(
                  fav => fav.name === pokemon.name
              )
          )
        : pokemons.value;

    if (!searchQuery.value) return source;

    return source.filter(pokemon =>
        pokemon.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
});
const displayedPokemons = computed(() => {
    const source = showFavoritePokemons.value
        ? filteredPokemons.value.filter(pokemon =>
              filteredPokemons.value.some(
                  fav => fav.name === pokemon.name
              )
          )
        : filteredPokemons.value;

    return source.slice(0, visibleCount.value);
});
const loadMore20 = () => {
    visibleCount.value += 20;
};
const loadMore50 = () => {
    visibleCount.value += 50;
};
const loadMore100000 = () => {
    visibleCount.value += 100000;
};
const hasMorePokemons = computed(() => {
    const total = showFavoritePokemons.value
        ? pokemons.value.filter(pokemon =>
              favoritePokemons.value.some(
                  fav => fav.name === pokemon.name
              )
          ).length
        : pokemons.value.length;

    return visibleCount.value < total;
});
watch(showFavoritePokemons, () => {
    visibleCount.value = 20;
});
const favoritePokemons = ref([]);

const savedFavorites = localStorage.getItem('favoritePokemons');

if (savedFavorites) {
    favoritePokemons.value = JSON.parse(savedFavorites);
}
const isFavorite = (pokemon) => {
    return favoritePokemons.value.some((fav) => fav.name === pokemon.name);
};
const toggleFavorite = (pokemon) => {
    const index = favoritePokemons.value.findIndex(
        fav => fav.name === pokemon.name
    );

    if (index === -1) {
        favoritePokemons.value.push(pokemon);
    } else {
        favoritePokemons.value.splice(index, 1);
    }
    
    localStorage.setItem(
        'favoritePokemons',
        JSON.stringify(favoritePokemons.value)
    );
};
onMounted(async () => {
  if (localStorage.getItem('pokemonInfo')) {
        pokemons.value = JSON.parse(localStorage.getItem('pokemonInfo'));
        console.log('Pokemons geladen uit localStorage');
            console.log(pokemons.value);
        loading.value = false;
        return;
    }
    try {
        const res = await fetch('https://pokeapi.co/api/v2/pokemon?limit=5000');
        if (!res.ok) throw new Error('Fout bij ophalen pokemons');
        const data = await res.json();
        pokemons.value = data.results;
        pokemons.value.forEach((pokemon) => {
            const pokemonId = pokemon.url.split('/')[6];
            const pokemonImgUrl = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemonId}.png`;
        });
        localStorage.setItem('pokemonInfo', JSON.stringify(pokemons.value));
    } catch (e) {
        console.error(e.message || 'Onbekende fout');
        error.value = e.message || 'Onbekende fout';
    } finally {
        loading.value = false;
    }
        console.log(pokemons.value);
    console.log('Pokemons geladen van API');
});
const selectedPokemon = ref(null);
const isSheetOpen = ref(false);

const pokemonDetails = ref(null);
const pokemonEvolution = ref([null]);
const openSheet = async (pokemon) => {
    const id = pokemon.url.split('/')[6];

    try {
        const res = await fetch(
            `https://pokeapi.co/api/v2/pokemon/${id}`
        );

        pokemonDetails.value = await res.json();

        selectedPokemon.value = pokemon;
        
    } catch (err) {
        console.error(err);
    }

      try {
    const speciesRes = await fetch(
        pokemonDetails.value.species.url
    );

    const speciesData = await speciesRes.json();

    const evolutionRes = await fetch(
        speciesData.evolution_chain.url
    );

    pokemonEvolution.value = await evolutionRes.json();

    isSheetOpen.value = true;
}
catch (err) {
    console.error(err);
}
    console.log(pokemonDetails.value);
    console.log(pokemonEvolution.value);
};

const closeSheet = () => {
    pokemonDetails.value = null;
    selectedPokemon.value = null;
    isSheetOpen.value = false;
};
const openPokemonByName = async (pokemonName) => {
    try {
        const res = await fetch(
            `https://pokeapi.co/api/v2/pokemon/${pokemonName}`
        );

        const pokemonData = await res.json();

        pokemonDetails.value = pokemonData;

        selectedPokemon.value = {
            name: pokemonData.name,
            url: `https://pokeapi.co/api/v2/pokemon/${pokemonData.id}/`
        };

        const speciesRes = await fetch(
            pokemonData.species.url
        );

        const speciesData = await speciesRes.json();

        const evolutionRes = await fetch(
            speciesData.evolution_chain.url
        );

        pokemonEvolution.value = await evolutionRes.json();
    }
    catch (err) {
        console.error(err);
    }
};
// Custom Install Banner logic
let deferredPrompt;
onMounted(() => {
  const installBanner = document.getElementById('installBanner');
  const installBtn = document.getElementById('installBtn');
  const closeBanner = document.getElementById('closeBanner');

  closeBanner?.addEventListener('click', () => {
    installBanner.style.display = 'none';
  });

  installBtn?.addEventListener('click', async () => {
    console.log('Install button clicked');
    if (deferredPrompt) {
      deferredPrompt.prompt();
      const { outcome } = await deferredPrompt.userChoice;

      if (outcome === 'accepted') {
        installBanner.style.display = 'none';
      }

      deferredPrompt = null;
    }
  });
});
window.addEventListener('appinstalled', () => {
  const installBanner = document.getElementById('installBanner');

  if (installBanner) {
    installBanner.style.display = 'none';
  }

  deferredPrompt = null;
});
window.addEventListener("load", () => {
  if ("serviceWorker" in navigator) {
    navigator.serviceWorker.register("../src/service-worker.js")
      .then(registration => {
        console.log("Service Worker registered with scope:", registration.scope);
      })
      .catch(error => {
        console.error("Service Worker registration failed:", error);
      });
  }
});
</script>
<template>
    <!-- Material.io bestanden (JS en registratie componenten) -->
      <!-- <link rel="manifest" href="../src/manifest.json"></link> -->
    <!-- <link src="https://unpkg.com/material-components-web@latest/dist/material-components-web.min.js"></link> -->
    <!-- <script>const MDCBanner = mdc.banner.MDCBanner;const MDCCheckbox = mdc.checkbox.MDCCheckbox;const MDCChip = mdc.chips.MDCChip;const MDCChipSet = mdc.chips.MDCChipSet;const MDCCircularProgress = mdc.circularProgress.MDCCircularProgress;const MDCDataTable = mdc.dataTable.MDCDataTable;const MDCDialog = mdc.dialog.MDCDialog;const MDCDrawer = mdc.drawer.MDCDrawer;const MDCFloatingLabel = mdc.floatingLabel.MDCFloatingLabel;const MDCFormField = mdc.formField.MDCFormField;const MDCIconButtonToggle = mdc.iconButton.MDCIconButtonToggle;const MDCLineRipple = mdc.lineRipple.MDCLineRipple;const MDCLinearProgress = mdc.linearProgress.MDCLinearProgress;const MDCList = mdc.list.MDCList;const MDCMenu = mdc.menu.MDCMenu;const MDCMenuSurface = mdc.menuSurface.MDCMenuSurface;const MDCNotchedOutline = mdc.notchedOutline.MDCNotchedOutline;const MDCRadio = mdc.radio.MDCRadio;const MDCRipple = mdc.ripple.MDCRipple;const MDCSegmentedButton = mdc.segmentedButton.MDCSegmentedButton;const MDCSelect = mdc.select.MDCSelect;const MDCSlider = mdc.slider.MDCSlider;const MDCSnackbar = mdc.snackbar.MDCSnackbar;const MDCSwitch = mdc.switchControl.MDCSwitch;const MDCTabBar = mdc.tabBar.MDCTabBar;const MDCTextField = mdc.textField.MDCTextField;const MDCTooltip = mdc.tooltip.MDCTooltip;const MDCTopAppBar = mdc.topAppBar.MDCTopAppBar;</script> -->


  <meta name="description" content="Schrijf hier de omschrijving over jouw App.">
<meta name="msapplication-TileColor" content="#55b4da">
<meta name="theme-color" content="#55b4da">
<link rel="icon" type="image/png" sizes="32x32" href="assets/img/icons/favicon-32x32.png">
    <!-- Material.io bestanden (CSS & Font/icons)-->
    <!-- Uitleg werking componenents: https://material.io/components?platform=web -->
    <!-- Demo van components:         https://material-components.github.io/material-components-web-catalog/#/ -->
    <!-- Demo en uitleg icons:        https://fonts.google.com/icons?selected=Material+Icons -->
    <link rel="stylesheet" href="https://unpkg.com/material-components-web@latest/dist/material-components-web.min.css">
    <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap">
  <!-- Custom Install Banner -->
  <div id="installBanner" style="display:none; position:fixed; left:50%; top:32px; transform:translateX(-50%); z-index:1000; background:#323232; color:white; border-radius:6px; box-shadow:0 2px 8px rgba(0,0,0,0.2); min-width:320px; max-width:90vw; padding:16px 16px 16px 20px; display:flex; align-items:center; gap:16px;">
    <span style="flex:1; font-size:1rem;">Wil je deze app op je apparaat installeren?</span>
    <button id="installBtn" style="background:#42a5f5; color:white; border:none; border-radius:4px; padding:8px 18px; font-weight:500; font-size:1rem; cursor:pointer;">INSTALLEREN</button>
    <button id="closeBanner" style="background:transparent; color:#bdbdbd; border:none; font-size:1.3rem; margin-left:8px; cursor:pointer;">&#10005;</button>
  </div>




    <header class="mdc-top-app-bar">
  <div class="mdc-top-app-bar__row">
    <section class="mdc-top-app-bar__section mdc-top-app-bar__section--align-start">
        <RouterLink to="/home">
          <button
            class="material-icons mdc-top-app-bar__navigation-icon mdc-icon-button"
            aria-label="Open navigation home"
          >home</button>
        </RouterLink>
      <span class="mdc-top-app-bar__title">Pokedex</span>
    </section>
<section class="mdc-top-app-bar__section mdc-top-app-bar__section--align-end" role="toolbar">
  <button class="material-icons mdc-top-app-bar__action-item mdc-icon-button" @click="showFavoritePokemons = !showFavoritePokemons">
    {{ showFavoritePokemons ? 'favorite' : 'favorite_border' }}
</button>
      <button class="material-icons mdc-top-app-bar__action-item mdc-icon-button" aria-label="Search">search</button>
      <section class="Searchbar" @clickshow="searchQuery = ''" @clickaway="searchQuery = ''">
        <input class="Searchbar__input" type="text" placeholder="Zoek een Pokémon..." v-model="searchQuery" @click.stop @click.away="searchQuery = ''">
      </section>
    </section>
  </div>
</header>
<main class="mdc-top-app-bar--fixed-adjust">
<div v-if="loading">Laden...</div>
    <div v-else-if="error" style="color: red">{{ error }}</div>
    <div v-else>
<div class="pokedex-grid">
      <h1 v-if="displayedPokemons.length === 0">Geen Pokémon gevonden</h1>
<div class="mdc-card" v-for="pokemon in displayedPokemons" :key="pokemon.name">
    <div class="mdc-card__primary-action" >
      <div class="mdc-card__media">
        <img class="pokemon-image" :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemon.url.split('/')[6]}.png`" :alt="pokemon.name">
      </div>
      <div class="pokemon-name">
        {{ pokemon.name }}
      </div>
      <div class="mdc-card__ripple"></div>
    </div>
    <div class="mdc-card__actions">
      <div class="mdc-card__action-buttons">
        <button class="mdc-button mdc-card__action mdc-card__action--button">
          <span class="mdc-button__label" @click="openSheet(pokemon)">
            Details
          </span>
    </button>
      </div>
      <div class="mdc-card__action-icons">
        <button class="material-icons mdc-icon-button mdc-card__action mdc-card__action--icon" @click="toggleFavorite(pokemon)">
          {{ isFavorite(pokemon) ? 'favorite' : 'favorite_border' }}
        </button>
      </div>
    </div>
  </div>
</div>
<section class="sheet" :class="{ 'sheet-out-of-view': !isSheetOpen }">
  <header class="sheet__topbar">
    <div class="sheet__topbar-left">
      <button class="sheet__icon-button material-icons" @click="closeSheet">
        close
      </button>
      <span class="sheet__title">
        {{ selectedPokemon?.name }}
      </span>
    </div>
  </header>

  <main class="sheet__hero">
    <img v-if="selectedPokemon" class="sheet__hero-image" :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${selectedPokemon.url.split('/')[6]}.png`" :alt="selectedPokemon.name">
  </main>

<div class="sheet__body">
  <div v-if="pokemonDetails" style="padding:20px;color:white;">
    <h1>{{ pokemonDetails.name }}</h1>

    <p><strong>ID:</strong> {{ pokemonDetails.id }}</p>
    <p><strong>Height:</strong> {{ pokemonDetails.height }}</p>
    <p><strong>Weight:</strong> {{ pokemonDetails.weight }}</p>
    <p><strong>Base Experience:</strong> {{ pokemonDetails.base_experience }}</p>
    <p><strong>Order:</strong> {{ pokemonDetails.order }}</p>
    <p><strong>Default:</strong> {{ pokemonDetails.is_default ? 'Ja' : 'Nee' }}</p>
    <h2>Types</h2>
    <ul>
      <li v-for="type in pokemonDetails.types" :key="type.type.name">
        {{ type.type.name }}
      </li>
    </ul>
    <h2>Abilities</h2>
    <ul>
      <li v-for="ability in pokemonDetails.abilities" :key="ability.ability.name" >
        {{ ability.ability.name }}
      </li>
    </ul>
    <h2>Stats</h2>
    <ul>
      <li v-for="stat in pokemonDetails.stats":key="stat.stat.name">
        {{ stat.stat.name }}:
        {{ stat.base_stat }}
      </li>
    </ul>
    <h2>Moves</h2>
    <ul>
      <li v-for="move in pokemonDetails.moves.slice(0, 30)" :key="move.move.name">
        {{ move.move.name }}
      </li>
    </ul>
    <h2>Sprites</h2>
    <div style=" display:grid; grid-template-columns:repeat(auto-fit,minmax(120px,1fr)); gap:10px; ">
      <img v-for="(sprite,key) in pokemonDetails.sprites" :key="key" v-show="typeof sprite === 'string' && sprite" :src="sprite" :alt="key" style="width:100%;background:white;" >
    </div>
<h2 v-if="pokemonEvolution?.chain">Evolution</h2>

<div class="evolution-chain" v-if="pokemonEvolution?.chain">
  <div v-for="species in [
      pokemonEvolution.chain.species,
      pokemonEvolution.chain.evolves_to[0]?.species,
      pokemonEvolution.chain.evolves_to[0]?.evolves_to[0]?.species
    ].filter(Boolean)"
    :key="species.name"
    class="evolution-item"
    :class="{ active: species.name === pokemonDetails.name }"
    @click="openPokemonByName(species.name)">
    <img :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${species.url.split('/')[6]}.png`":alt="species.name">

    <p>{{ species.name }}</p>
  </div>
</div>
  </div>
</div>

  <footer class="sheet__footer">
    <button class="sheet__footer-button material-icons" @click="toggleFavorite(selectedPokemon)" v-if="selectedPokemon" >
      {{ isFavorite(selectedPokemon) ? 'favorite' : 'favorite_border'}}
    </button>
  </footer>
</section>
    </div>
    <div class="load-more-container" v-if="hasMorePokemons && displayedPokemons.length >= visibleCount">
    <button class="mdc-button mdc-button--raised" @click="loadMore20" >
        <span class="mdc-button__label">
             +20
        </span>
    </button>
      <span style="margin: 0 12px; font-weight: 500;"></span>
        <button class="mdc-button mdc-button--raised" @click="loadMore50" >
        <span class="mdc-button__label">
            +50
        </span>
    </button>
          <span style="margin: 0 12px; font-weight: 500;"></span>
        <button class="mdc-button mdc-button--raised" @click="loadMore100000" >
        <span class="mdc-button__label">
            All
        </span>
    </button>
</div>
</main>
    
    </template>
    <style>
    .evolution-chain{
  display:flex;
  gap:20px;
  flex-wrap:wrap;
}

.evolution-item{
  border:3px solid transparent;
  border-radius:12px;
  padding:10px;
  text-align:center;
}

.evolution-item.active{
  border:3px solid limegreen;
}

.evolution-item img{
  width:100px;
  height:100px;
}
/* Mobile first */
body {
  margin: 0;
}
.pokedex-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 16px;
    padding: 16px;
}

.mdc-card {
    width: 100%;
    margin: 0;
}

.mdc-card__media {
    height: 180px;
    display: flex;
    justify-content: center;
    align-items: center;
    background: #f5f5f5;
}

.pokemon-image {
    width: 120px;
    height: 120px;
    object-fit: contain;
}

.pokemon-name {
    text-align: center;
    padding: 12px;
    font-size: 1.1rem;
    font-weight: 500;
    text-transform: capitalize;
}

.mdc-card__actions {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.mdc-top-app-bar__title {
    font-size: 1.1rem;
}

/* Tablet */
@media (min-width: 600px) {
    .pokedex-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

/* Kleine laptop */
@media (min-width: 900px) {
    .pokedex-grid {
        grid-template-columns: repeat(3, 1fr);
    }
}

/* Desktop */
@media (min-width: 1200px) {
    .pokedex-grid {
        grid-template-columns: repeat(4, 1fr);
    }
}
.load-more-container {
    display: flex;
    justify-content: center;
    padding: 24px;
}
        .hidden{
          display: none;
        }


.sheet {
  position: fixed;
  inset: 0;

  width: 100vw;
  height: 100%;

  display: flex;
  flex-direction: column;

  background: black;

  z-index: 9999;

  overflow: auto;
}
.sheet-out-of-view {
  transform: translateY(100%);
  opacity: 0;
}

.sheet__topbar {
  height: 48px;
  background:  rgb(255, 111, 0);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 12px;
  color: white;
  flex-shrink: 0;
}

.sheet__topbar-left,
.sheet__topbar-right {
  display: flex;
  align-items: center;
  gap: 4px;
}

.sheet__icon-button {
  border: 0;
  background: transparent;
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: grid;
  place-items: center;
  font-size: 22px;
}

.sheet__title {
  font-size: 18px;
  font-weight: 500;
}
.sheet__hero {
  flex: 0 0 180px;

  background: #e7ecef;
  display: flex;
  justify-content: center;
  align-items: center;

  padding: 12px;
  flex-shrink: 0;
}

.sheet__hero-image {
  max-height: 150px;
  width: auto;
  max-width: 100%;
}

.sheet__body {
  flex: 1 1 auto;
  min-height: 0;
  background: #000000;
}

.sheet__footer {
  flex-shrink: 0;

  height: 50px;

  background: rgb(255, 111, 0);

  display: flex;
  justify-content: center;
  align-items: center;
}

.sheet__footer-button {
  border: 0;
  background: transparent;
  color: white;
  width: 48px;
  height: 44px;
  font-size: 22px;
}

.sheet {
  position: fixed;
  inset: 0;

  display: flex;
  flex-direction: column;

  width: 100vw;
  height: 100vh;

  background: black;
  z-index: 9999;
}

.sheet__topbar {
  flex-shrink: 0;
}

.sheet__hero {
  flex-shrink: 0;
}

.sheet__body {
  flex: 1;
  overflow-y: auto;

  background: #2e2e2e;
  color: white;

  padding: 16px;
}

.sheet__footer {
  position: sticky;
  bottom: 0;

  height: 50px;

  background: rgb(255, 111, 0);

  display: flex;
  justify-content: center;
  align-items: center;

  z-index: 10000;
}
     </style>