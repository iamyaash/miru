<script lang='ts'>
  import { Toaster as Sonner, type ToasterProps as SonnerProps } from 'svelte-sonner'

  type $$Props = SonnerProps

</script>

<Sonner
  theme='dark'
  class='toaster group'
  toastOptions={{
    classes: {
      toast: 'group toast group-[.toaster]:bg-background group-[.toaster]:text-foreground group-[.toaster]:border-border group-[.toaster]:shadow-lg',
      description: 'group-[.toast]:text-muted-foreground whitespace-pre-line',
      actionButton: 'group-[.toast]:bg-primary group-[.toast]:text-primary-foreground',
      cancelButton: 'group-[.toast]:bg-muted group-[.toast]:text-muted-foreground'
    }
  }}
  {...$$restProps}
/>
