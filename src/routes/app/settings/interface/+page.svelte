<script lang='ts' context='module'>
  // eslint-disable-next-line svelte/no-svelte-internal
  import { append, element } from 'svelte/internal'
  import { persisted } from 'svelte-persisted-store'

  const style = element('style')
  style.id = 'customThemes'
  append(document.head, style)

  export const variables = persisted('theme', '')

  variables.subscribe(value => {
    style.textContent = `:root{${value.replace(/{|}/g, '')}}`
  })
</script>

<script lang='ts'>
  import SettingCard from '$lib/components/SettingCard.svelte'
  import { SingleCombo } from '$lib/components/ui/combobox'
  import { Input } from '$lib/components/ui/input'
  import { Switch } from '$lib/components/ui/switch'
  import { Textarea } from '$lib/components/ui/textarea'
  import native from '$lib/modules/native'
  import { settings, SUPPORTS } from '$lib/modules/settings'

  const angle = {
    default: 'Default',
    d3d9: 'D3D9',
    d3d11: 'D3D11',
    warp: 'Warp',
    gl: 'GL',
    gles: 'GLES',
    swiftshader: 'SwiftShader',
    vulkan: 'Vulkan',
    metal: 'Metal'
  }
</script>

<div class='space-y-3 pb-10 lg:max-w-4xl'>
  {#if !SUPPORTS.isAndroid}
    <div class='font-weight-bold text-xl font-bold'>Rich Pressence Settings</div>
    <SettingCard let:id title='Show Details in Discord Rich Presence' description='Shows currently played anime and episode in Discord rich presence.'>
      <Switch {id} bind:checked={$settings.showDetailsInRPC} />
    </SettingCard>
  {/if}
  <SettingCard let:id title='CSS Variables' description='Used for custom themes. Can change colors, sizes, spacing and more. Supports only variables. Best way to discover variables is to use the built-in devtools via Ctrl+Shift+I or F12.'>
    <Textarea class='form-control w-60 shrink-0 mw-full bg-dark' placeholder='--accent-color: #e5204c;' bind:value={$variables} {id} />
  </SettingCard>
  {#if !SUPPORTS.isAndroid}
    <div class='font-weight-bold text-xl font-bold'>Rendering Settings</div>
    <SettingCard title='ANGLE Backend' description="What ANGLE backend to use for rendering. DON'T CHANGE WITHOUT REASON! On some Windows machines D3D9 might help with flicker. Changing this setting to something your device doesn't support might prevent Hayase from opening which will require a full reinstall. While Vulkan is an available option it might not be fully supported on Linux.">
      <SingleCombo bind:value={$settings.angle} items={angle} class='w-40 shrink-0 border-input border' />
    </SettingCard>

    <div class='font-weight-bold text-xl font-bold'>UI Settings</div>
    <SettingCard title='Idle Animation' description='Enable/Disable the 3d idle animation. Changing this setting will restart the app.' let:id>
      <Switch bind:checked={$settings.idleAnimation} on:click={native.restart} {id} />
    </SettingCard>
  {:else}
    <div class='font-weight-bold text-xl font-bold'>UI Settings</div>
  {/if}
  <SettingCard title='UI Scale' description='Change the zoom level of the interface.' let:id>
    <Input type='number' inputmode='numeric' pattern='[0-9]*' min='0' max='65536' bind:value={$settings.uiScale} {id} class='w-32 shrink-0 bg-background' />
  </SettingCard>
</div>
