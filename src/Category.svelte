<script>
  import Item from './Item.svelte';
  import Dialog from './Dialog.svelte';
  import {getGuid, blurOnKey, sortOnName} from './util';
  import {flip} from 'svelte/animate';
  import {createEventDispatcher} from 'svelte';
  import {linear} from 'svelte/easing';
  import {scale} from 'svelte/transition';

  export let categories;
  export let category;
  export let show;
  export let dnd;

  let editing = false;
  let itemName = '';
  let items = [];
  let message = '';
  let dialog = null;
  let hovering = false;
  const dispatch = createEventDispatcher();
  const options = {duration: 700, easing: linear, times: 2};

  $: items = Object.values(category.items);
  $: remaining = items.filter(item => !item.packed).length;
  $: total = items.length;
  $: status = `${remaining} of ${total} remaining`;
  $: itemsToShow = sortOnName(items.filter(i => shouldShow(show, i)));

  function addItem() {
    const duplicate = Object.values(categories).some(cat =>
    Object.values(cat.items).some(item => item.name === itemName)
    );
    if (duplicate) {
      message = `The item "${itemName}" already exists.`;
      dialog.showModal();
      return;
    }
    const {items} = category;
    const id = getGuid();
    items[id] = {id, name: itemName, packed: false};
    category.items = items;
    itemName = '';
  }

  function deleteItem(item) {
    delete category.items[item.id];
    category = category;
  }

  function shouldShow(show, item) {
    return (
      show === 'all' ||
      (show === 'packed' && item.packed) ||
      (show === 'unpacked' && !item.packed)
    );
  }

  function spin(node, options) {
    const {easing, times = 1} = options;
    return {
      ...options,
      css(t) {
      const eased = easing(t);
      const degrees = 360 * times;
      return 'transform-origin: 50% 50%; ' +
      `transform: scale(${eased}) ` +
      `rotate(${eased * degrees}deg);`;
      }
    };
  }
</script>

<section class:hover={hovering}
  out:spin={options}
  in:scale={options}
  on:dragenter={() => (hovering = true)}
  on:dragleave={event => {
    const {localName} = event.target;
    if (localName === 'section') hovering = false;
  }}
  on:drop|preventDefault={event => {
    dnd.drop(event, category.id);
    hovering = false;
  }}
  on:dragover|preventDefault>
  <h3>
    {#if editing}
      <input
        bind:value={category.name}
        on:blur={() => (editing = false)}
        on:keypress={blurOnKey} />
      {:else}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <span on:click={() => (editing = true)}>{category.name}</span>
    {/if}
    <span class="status">{status}</span>
    <button class="icon" on:click={() => dispatch('delete')}>
      &#x1F5D1;
    </button>
  </h3>
  <form on:submit|preventDefault={addItem}>
    <label>
      New Item
      <input data-testid="item-input" required bind:value={itemName}>
    </label>
    <button disabled={!itemName}>Add Item</button>
  </form>
  <ul>
    {#each itemsToShow as item (item.id)}
    <div animate:flip>
      <Item categoryId={category.id} bind:item on:delete={() => deleteItem(item)} {dnd}/>
    </div>
    {:else}
      <div>This category does not contain any items yet.</div>
    {/each}
  </ul>
  <Dialog title="Category" bind:dialog>
    <div>{message}</div>
  </Dialog>
</section>

<style>
  button,
  input {
    border: solid lightgray 1px;
  }
  button.icon {
    border: none;
  }
  .hover {
  border-color: orange;
  }
  h3 {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin: 0;
  }
  section {
    --padding: 10px;
    background-color: white;
    border: solid transparent 3px;
    border-radius: var(--padding);
    color: black;
    display: inline-block;
    margin: var(--padding);
    padding: calc(var(--padding) * 2);
    padding-top: var(--padding);
    vertical-align: top;
  }
  .status {
    font-size: 18px;
    font-weight: normal;
    margin: 0 15px;
  }
  ul {
    list-style: none;
    margin: 0;
    padding-left: 0;
  }
</style>