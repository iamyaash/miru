<script lang='ts'>
  import { DropdownMenu as DropdownMenuPrimitive } from 'bits-ui'

  import { cn } from '$lib/utils.js'

  type $$Props = DropdownMenuPrimitive.SeparatorProps

  let className: $$Props['class'] = undefined
  export { className as class }
</script>

<DropdownMenuPrimitive.Separator
  class={cn('bg-muted -mx-1 my-1 h-px', className)}
  {...$$restProps}
/>
