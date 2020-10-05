<script>
  import { scale } from "svelte/transition";
  import { quintOut } from "svelte/easing";

  const regExpEscape = s => {
    return s.replace(/[-\\^$*+?.()|[\]{}]/g, "\\$&");
  };

  export let asyncFunc = null;
  export let id = "";
  export let items = [];
  export let maxItems = 6;
  export let minChar = 3;
  export let name = "";
  export let results = [];
  export let value = {};

  let arrowCounter = 0;
  let input;
  let loading = false;
  let open = false;
  let search = "";
  let timer = null;

  const clickAway = event => {
    timer = setTimeout(() => {
      open = false;
    }, 200);
  };

  const clickOn = event => clearTimeout(timer);

  const onChange = event => {
    const term = event.target.value;

    if (term.length >= Number(minChar)) {
      if (asyncFunc !== null) {
        loading = true;

        clearTimeout(timer);

        timer = setTimeout(async () => {
          items = await asyncFunc(term);
          loading = false;
          open = true;
          filterResults(term);
        }, 500);
      } else {
        open = true;
        filterResults(term);
      }
    } else if (term.length === 0) {
      value = {};
    } else {
      results = [];
    }
  };

  const filterResults = term => {
    results = (items || [])
      .filter(item => {
        if (typeof item !== "string") item = item.value || "";

        return item.toUpperCase().includes(term.toUpperCase());
      })
      .map((item, i) => {
        const text = typeof item !== "string" ? item.value : item;

        return {
          key: item.key || i,
          value: text,
          label:
            term.trim() === ""
              ? text
              : text.replace(
                  RegExp(regExpEscape(term.trim()), "i"),
                  "<span style='font-weight: 700;'>$&</span>"
                )
        };
      })
      .slice(0, maxItems - 1);
  };

  const onKeyDown = event => {
    if (event.keyCode === 40 && arrowCounter < results.length - 1) {
      arrowCounter++;
    } else if (event.keyCode === 38 && arrowCounter > 0) {
      arrowCounter--;
    } else if (event.keyCode === 13) {
      event.preventDefault();

      if (arrowCounter === -1) arrowCounter = 0;

      close(arrowCounter);
    } else if (event.keyCode === 27) {
      event.preventDefault();
      close();
    }
  };

  const close = (index = -1) => {
    open = false;
    arrowCounter = -1;
    input.blur();

    if (index > -1) {
      value = results[index];
      search = results[index].value;
    } else {
      search = "";
    }
  };
</script>

<div>
  <input
    on:blur={clickAway}
    on:focus={clickOn}
    on:keydown={event => onKeyDown(event)}
    on:input={event => onChange(event)}
    bind:value={search}
    bind:this={input}
    class="input"
    type="text"
    autocomplete="off"
    {id}
    {name} />
  {#if loading}
    <!-- <div
      style="position: absolute; top: 0;
      bottom: 0; right: 0; padding-right: 0.75rem; flex: 1 1 0%; align-items: center; pointer-events: none;">
      <img style="height: 1rem; width: 1rem; opacity: 0.5;" src="./25.svg" alt="Loading" />
    </div> -->
  {/if}
  {#if open}
    <div
      class="results-container"
      transition:scale={{
        duration: 200,
        delay: 0,
        opacity: 0.2,
        start: 0.0,
        easing: quintOut
      }}
    >
      <div class="result-container-box">
        <div class="result-container-box-position">
          {#if results.length === 0}
            <div
              class="results">
              No matches
            </div>
          {:else}
            {#each results as result, i}
              <div
                on:click={() => close(i)}
                class="results">
                {@html result.label}
              </div>
            {/each}
          {/if}
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
  .result-container-box-position {
    padding-top: 0.25rem;
    padding-bottom: 0.25rem;
  }
  .result-container-box {
    border-radius: 0.375rem;
    background-color: #fff;
    box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.05);
  }
  .results-container {
    transform-origin: top right;
    position: absolute;
    right: 10;
    margin-top: 0.5rem;
    width: 87%;
    border-radius: 0.375rem;
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
    z-index: 10;
    cursor: pointer;
  }
  .results {
    display: block;
    padding-left: 1rem;
    padding-right: 1rem;
    padding-top: 0.5rem;
    padding-bottom: 0.5rem;
    font-size: 0.875rem;
    cursor: pointer;
    color: #718096;
    background-color: #f7fafc;
  }
  .results:hover {
    background-color: #edf2f7;
  }
</style>