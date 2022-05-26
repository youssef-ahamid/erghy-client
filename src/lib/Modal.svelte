<svelte:options accessors={true} />

<script>
  import { fade, fly } from 'svelte/transition'
  export let active = false
  export let key = 0

  export const activate = () => {
    active = true
    key++
  }
</script>

{#if active}
  {#key key}
    <div
      class="fixed inset-0 z-[999] flex h-screen w-screen items-center justify-center bg-black bg-opacity-75"
      transition:fade={{ duration: 300 }}
    >
      <div
        class="max-h-fit max-w-fit rounded-3xl bg-white p-12 text-black md:max-w-xl md:rounded-[50px] md:p-20 relative"
        id="modal"
        transition:fly={{ duration: 300, y: 50, delay: 200 }}
      >
        <button class="absolute right-0 top-0 translate-x-1/2 -translate-y-1/2 rounded-full min-w-[40px] min-h-[40px] max-w-[40px] max-h-[40px] bg-white shadow-lg flex justify-center items-center" on:click={() => {active = false}}>
            <p class="leading-none text-lg font-bold text-black text-center">x</p>
        </button>
        <slot />
      </div>
    </div>
  {/key}
{/if}
