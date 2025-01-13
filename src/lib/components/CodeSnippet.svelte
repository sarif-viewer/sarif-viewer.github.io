<script lang="ts">
	let { code, line }: { code: string | undefined; line: number | undefined } = $props();
</script>

<div
	class="relative rounded-md bg-slate-100 dark:bg-[#2d2d2d] border border-slate-200 dark:border-slate-700 p-4 font-mono text-sm overflow-hidden"
>
	<div class="flex items-start">
		<!-- Line number -->
		<div
			class="text-slate-500 select-none pr-4 border-r border-slate-200 dark:border-slate-700 min-w-[3rem] text-right"
		>
			{line}
		</div>
		<!-- Code content -->
		<pre class="overflow-x-auto pl-4 m-0"><code
				class="block whitespace-pre text-slate-800 dark:text-slate-200">{code}</code
			></pre>
	</div>
</div>

<style>
	pre {
		margin: 0;
		padding: 0;
		background: transparent !important;
	}

	code {
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono',
			'Courier New', monospace;
	}
</style>
