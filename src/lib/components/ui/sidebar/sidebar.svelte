<script lang='ts'>
  import Menu from 'lucide-svelte/icons/menu'
  import X from 'lucide-svelte/icons/x'

  import { Button } from '../button'

  import { onNavigate } from '$app/navigation'
  import { breakpoints } from '$lib/utils'

  let open = false // 152 x 140

  onNavigate(() => {
    open = false
  })

  let container: HTMLDivElement | undefined

  function outsideclick (node: HTMLDivElement) {
    const ctrl = new AbortController()

    node.addEventListener('click', e => {
      if (!container || container.contains(e.target as Node)) return
      open = false
    }, { signal: ctrl.signal })

    return { destroy: () => ctrl.abort() }
  }
</script>

<svelte:window use:outsideclick />

{#if !$breakpoints.md}
  <div class='shrink-0 z-50 bg-black absolute left-4 bottom-4 w-14 h-[52px] flex rounded-md items-end justify-end overflow-clip transition-[width,height] group-fullscreen/fullscreen:hidden' class:!w-[152px]={open} class:!h-[140px]={open} bind:this={container}>
    <div class='p-2 grid grid-cols-3 gap-2 shrink-0'>
      <slot />
      <Button variant='ghost' class='px-2 w-full relative' on:click={() => { open = !open }}>
        {#if open}
          <X size={18} fill='currentColor' class='pointer-events-none' />
        {:else}
          <Menu size={18} fill='currentColor' class='pointer-events-none' />
        {/if}
      </Button>
    </div>
  </div>
{:else}
  <div class='w-14 p-2 md:pl-0 flex flex-col z-10 shrink-0 bg-black gap-2 group-fullscreen/fullscreen:hidden'>
    <slot />
  </div>
{/if}
