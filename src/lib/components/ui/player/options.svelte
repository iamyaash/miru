<script lang='ts'>
  import EllipsisVertical from 'lucide-svelte/icons/ellipsis-vertical'
  import { getContext, tick } from 'svelte'

  import { Input } from '../input'

  import Keybinds from './keybinds.svelte'
  import { normalizeSubs, normalizeTracks, type Chapter } from './util'

  import type PictureInPicture from './pip'
  import type { ResolvedFile } from './resolver'
  import type Subtitles from './subtitles'
  import type { Writable } from 'simple-store-svelte'
  import type { HTMLAttributes } from 'svelte/elements'

  import { beforeNavigate } from '$app/navigation'
  import { Button } from '$lib/components/ui/button'
  import * as Dialog from '$lib/components/ui/dialog'
  import * as Tooltip from '$lib/components/ui/tooltip'
  import * as Tree from '$lib/components/ui/tree'
  import { dragScroll, keywrap } from '$lib/modules/navigate'
  import { settings } from '$lib/modules/settings'
  import { cn, toTS } from '$lib/utils'

  export let wrapper: HTMLDivElement

  export let video: HTMLVideoElement

  export let selectAudio: (id: string) => void
  export let selectVideo: (id: string) => void
  export let fullscreen: () => void
  export let chapters: Chapter[]
  export let seekTo: (time: number) => void
  export let playbackRate: number
  export let subtitles: Subtitles | undefined
  export let videoFiles: ResolvedFile[]
  export let selectFile: (file: ResolvedFile) => void
  export let pip: PictureInPicture
  export let subtitleDelay: number

  $: pipElement = pip.element

  $: tracks = subtitles?._tracks
  $: current = subtitles?.current

  let open = false

  let treeState: Writable<string[]>

  export async function openSubs () {
    open = true
    await tick()
    treeState.set(['subs'])
  }

  let className: HTMLAttributes<HTMLDivElement>['class'] = ''
  export { className as class }

  export let showKeybinds = false
  function close () {
    if (showKeybinds) {
      showKeybinds = false
    } else {
      open = false
    }
  }
  function deband () {
    $settings.playerDeband = !$settings.playerDeband
  }

  let fullscreenElement: HTMLElement | null = null

  export let id = ''

  let keybindDesc: unknown = null

  const stopProgressBar = getContext<() => void>('stop-progress-bar')
  beforeNavigate(({ cancel }) => {
    if (open) {
      open = false
      cancel()
      stopProgressBar()
    }
  })
</script>

<Dialog.Root portal={wrapper} bind:open>
  <Dialog.Trigger asChild let:builder>
    <Button class={cn('p-3 w-12 h-12', className)} variant='ghost' builders={[builder]} on:keydown={keywrap(() => { open = !open })} data-left='#player-volume-button, #player-next-button, #player-prev-button, #player-play-pause-button' data-up='#player-seekbar' {id}>
      <EllipsisVertical size='24px' class='p-[1px]' />
    </Button>
  </Dialog.Trigger>
  <Dialog.Content class='absolute bg-transparent border-none p-0 shadow-none size-full overflow-hidden'>
    <div on:pointerdown|self={close} class='size-full flex justify-center items-center flex-col overflow-y-scroll text-[6px] lg:text-xs' use:dragScroll>
      {#if showKeybinds}
        <div class='bg-black py-3 px-4 rounded-md text-sm lg:text-lg font-bold mb-4'>
          {keybindDesc ?? 'Drag and drop binds to change them'}
        </div>
        <Keybinds let:prop={item} autosave={true} clickable={true} on:pointerleave={() => { keybindDesc = null }} pointerOver={item => { keybindDesc = item?.desc }}>
          {#if item?.type}
            <div class='size-full flex justify-center p-1.5 lg:p-3'>
              {#if item.icon}
                <svelte:component this={item.icon} size='2rem' class='h-full' fill={item.id === 'play_arrow' ? 'currentColor' : 'none'} />
              {/if}
            </div>
          {:else}
            <div class='size-full content-center text-center lg:text-lg'>{item?.id ?? ''}</div>
          {/if}
        </Keybinds>
      {:else}
        <Tree.Root bind:state={treeState}>
          {#if 'audioTracks' in HTMLVideoElement.prototype}
            <Tree.Item>
              <span slot='trigger'>Audio</span>
              <Tree.Sub>
                {#each Object.entries(normalizeTracks(video.audioTracks ?? [])) as [lang, tracks] (lang)}
                  <Tree.Item>
                    <span slot='trigger' class='capitalize'>{lang}</span>
                    <Tree.Sub>
                      {#each tracks as track (track.id)}
                        <Tree.Item active={track.enabled} on:click={() => { selectAudio(track.id); open = false }}>
                          <span>{track.label}</span>
                        </Tree.Item>
                      {/each}
                    </Tree.Sub>
                  </Tree.Item>
                {/each}
              </Tree.Sub>
            </Tree.Item>
          {/if}
          {#if 'videoTracks' in HTMLVideoElement.prototype}
            <Tree.Item>
              <span slot='trigger'>Video</span>
              <Tree.Sub>
                {#each Object.entries(normalizeTracks(video.videoTracks ?? [])) as [lang, tracks] (lang)}
                  <Tree.Item>
                    <span slot='trigger' class='capitalize'>{lang}</span>
                    <Tree.Sub>
                      {#each tracks as track (track.id)}
                        <Tree.Item active={track.enabled} on:click={() => { selectVideo(track.id); open = false }}>
                          <span>{track.label}</span>
                        </Tree.Item>
                      {/each}
                    </Tree.Sub>
                  </Tree.Item>
                {/each}
              </Tree.Sub>
            </Tree.Item>
          {/if}
          {#if subtitles}
            <Tree.Item id='subs'>
              <span slot='trigger'>Subtitles</span>
              <Tree.Sub>
                <Tree.Item active={Number($current) === -1} on:click={() => { $current = -1; open = false }}>
                  <span>OFF</span>
                </Tree.Item>
                {#each Object.entries(normalizeSubs($tracks)) as [lang, tracks] (lang)}
                  <Tree.Item>
                    <span slot='trigger' class='capitalize'>{lang}</span>
                    <Tree.Sub>
                      {#each tracks as { number, name }, i (i)}
                        <Tree.Item active={Number(number) === Number($current)} on:click={() => { $current = number; open = false }}>
                          <span>{name}</span>
                        </Tree.Item>
                      {/each}
                    </Tree.Sub>
                  </Tree.Item>
                {/each}
                <Tree.Item on:click={() => { subtitles.pickFile(); open = false }}>
                  <span>Add Subtitle File</span>
                </Tree.Item>
                <div class='flex items-center relative scale-parent font-bold'>
                  <div class='shrink-0 absolute left-4 z-10 pointer-events-none text-sm leading-5'>Delay</div>
                  <Input type='number' inputmode='numeric' pattern='[0-9]*.?[0-9]*' step='0.1' bind:value={subtitleDelay} {id} class='w-full shrink-0 px-12 border-0 !ring-0 no-scale text-right hover:bg-accent hover:text-accent-foreground rounded-sm' />
                  <div class='shrink-0 absolute right-3 z-10 pointer-events-none text-sm leading-5'>sec</div>
                </div>
              </Tree.Sub>
            </Tree.Item>
          {/if}
          <Tree.Item>
            <span slot='trigger'>Chapters</span>
            <Tree.Sub>
              {#each chapters as { text, start }, i (i)}
                <Tree.Item on:click={() => { seekTo(start); open = false }}>
                  <div class='flex justify-between w-full pr-2'>
                    <span>{text || '?'}</span>
                    <span class='text-muted-foreground'>{toTS(start || 0)}</span>
                  </div>
                </Tree.Item>
              {/each}
            </Tree.Sub>
          </Tree.Item>
          <Tree.Item>
            <span slot='trigger'>Playback Rate</span>
            <Tree.Sub>
              <Tree.Item on:click={() => { playbackRate = 0.5; open = false }}>
                <span>0.5x</span>
              </Tree.Item>
              <Tree.Item on:click={() => { playbackRate = 0.75; open = false }}>
                <span>0.75x</span>
              </Tree.Item>
              <Tree.Item on:click={() => { playbackRate = 1; open = false }}>
                <span>1x</span>
              </Tree.Item>
              <Tree.Item on:click={() => { playbackRate = 1.25; open = false }}>
                <span>1.25x</span>
              </Tree.Item>
              <Tree.Item on:click={() => { playbackRate = 1.5; open = false }}>
                <span>1.5x</span>
              </Tree.Item>
              <Tree.Item on:click={() => { playbackRate = 1.75; open = false }}>
                <span>1.75x</span>
              </Tree.Item>
              <Tree.Item on:click={() => { playbackRate = 2; open = false }}>
                <span>2x</span>
              </Tree.Item>
            </Tree.Sub>
          </Tree.Item>
          <Tree.Item>
            <span slot='trigger'>Playlist</span>
            <Tree.Sub class='w-auto max-w-96'>
              {#each videoFiles as file, i (i)}
                <Tree.Item on:click={() => selectFile(file)}>
                  <Tooltip.Root>
                    <Tooltip.Trigger class='text-ellipsis text-nowrap overflow-clip w-full' tabindex={-1}>
                      {file.name}
                    </Tooltip.Trigger>
                    <Tooltip.Content>
                      {file.name}
                    </Tooltip.Content>
                  </Tooltip.Root>
                </Tree.Item>
              {/each}
            </Tree.Sub>
          </Tree.Item>
          <Tree.Item on:click={() => (showKeybinds = !showKeybinds)}>
            Keybinds
          </Tree.Item>
          <Tree.Item on:click={fullscreen} active={!!fullscreenElement}>
            Fullscreen
          </Tree.Item>
          <Tree.Item on:click={() => { pip.pip(); close() }} active={!!$pipElement}>
            Picture in Picture
          </Tree.Item>
          <Tree.Item on:click={deband} active={$settings.playerDeband}>
            Deband
          </Tree.Item>
        </Tree.Root>
      {/if}
    </div>
  </Dialog.Content>
</Dialog.Root>

<svelte:document bind:fullscreenElement />
