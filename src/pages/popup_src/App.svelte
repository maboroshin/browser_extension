<script>
  const browser = window.browser || window.chrome

  import utils from "../../assets/javascripts/utils.js"
  import { onDestroy } from "svelte"
  import servicesHelper from "../../assets/javascripts/services.js"
  import { onMount } from "svelte"
  import Buttons from "./Buttons.svelte"

  import { options, config, page } from "./stores"

  let _options
  const unsubscribeOptions = options.subscribe(val => {
    if (val) {
      _options = val
      browser.storage.local.set({ options: val })
    }
  })

  let _config
  const unsubscribeConfig = config.subscribe(val => (_config = val))

  onDestroy(() => {
    unsubscribeOptions()
    unsubscribeConfig()
  })

  onMount(async () => {
    let opts = await utils.getOptions()
    if (!opts) {
      await servicesHelper.initDefaults()
      opts = await utils.getOptions()
    }
    options.set(opts)
    config.set(await utils.getConfig())
  })

  let _page
  page.subscribe(val => (_page = val))

  const dark = {
    text: "#fff",
    bgMain: "#121212",
    bgSecondary: "#202020",
    active: "#fbc117",
    danger: "#f04141",
    lightGrey: "#c3c3c3",
  }
  const light = {
    text: "black",
    bgMain: "white",
    bgSecondary: "#e4e4e4",
    active: "#fb9817",
    danger: "#f04141",
    lightGrey: "#c3c3c3",
  }
  let cssVariables
  $: if (_options) {
    if (_options.theme == "dark") {
      cssVariables = dark
    } else if (_options.theme == "light") {
      cssVariables = light
    } else if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
      cssVariables = dark
    } else {
      cssVariables = light
    }
  }
</script>

{#if _options && _config}
  <div
    class="main"
    dir="auto"
    style="
    --text: {cssVariables.text};
    --bg-main: {cssVariables.bgMain};
    --bg-secondary: {cssVariables.bgSecondary};
    --active: {cssVariables.active};
    --danger: {cssVariables.danger};
    --light-grey: {cssVariables.lightGrey};"
  >
    <Buttons />
  </div>
{:else}
  <p>Loading...</p>
{/if}

<style>
  :global(html, body) {
    min-width: 260px;
    height: min-content;
    min-height: auto;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  :global(body) {
    margin-top: -20px;
  }

  div {
    font-weight: bold;
    height: 100%;
    margin: 0;
    padding: 10px;
    padding-top: 20px;
    font-family: "Inter", sans-serif;
    font-size: 16px;
    background-color: var(--bg-main);
    color: var(--text);
  }
</style>
