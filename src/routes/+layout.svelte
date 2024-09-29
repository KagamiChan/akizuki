<script>
  import Background from '$lib/components/background.svelte'
  import '../app.css'

  import { ModeWatcher } from 'mode-watcher'
</script>

<ModeWatcher />
<Background />
<slot></slot>
