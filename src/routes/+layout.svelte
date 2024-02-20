<script>
	import "../app.pcss";
	import { Toaster } from "$lib/components/ui/sonner";
	import { ModeWatcher } from "mode-watcher";
</script>

<svelte:head>
	<title>GPA Calculator</title>
</svelte:head>

<ModeWatcher />

<Toaster />

<slot />
