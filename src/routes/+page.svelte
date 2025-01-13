<script lang="ts">
	import { AlertTriangle, CheckCircle, XCircle, Info, Upload } from "lucide-svelte";
	import CodeSnippet from "$lib/components/CodeSnippet.svelte"
	import { fade, slide } from "svelte/transition";
	import type { Log, Result, Tool } from 'sarif';

	// Create state for results and loading state
	let results: Result[] = $state([]);
	let tool: Tool = [];
	let isLoading: boolean = $state(false);
	let error: string = $state("");

	/** Handle file upload and parse SARIF */
	const handleFileUpload = async (event: Event) => {
		const target = event.target as HTMLInputElement;
		const file = target.files?.[0];

		if (!file) return;

		// Reset states
		error = "";
		isLoading = true;


		const fileContent = await file.text();
		const sarifData: Log = JSON.parse(fileContent);
		results = sarifData.runs[0].results ?? [];
		tool = sarifData.runs[0].tool ?? [];
		isLoading = false;
	};

	// Get severity icon based on level
	const getSeverityIcon = (level: Result.level | undefined) => {
		switch (level) {
			case "error":
				return XCircle;
			case "warning":
				return AlertTriangle;
			case "note":
				return Info;
			default:
				return CheckCircle;
		}
	};

	// Get severity color based on level
	const getSeverityColor = (level: Result.level | undefined) => {
		switch (level) {
			case "error":
				return "text-red-500";
			case "warning":
				return "text-yellow-500";
			case "note":
				return "text-blue-500";
			default:
				return "text-green-500";
		}
	};
	function getLevel(result: Result) {
		if (result.level) {
			return result.level;
		}
		for (const rule of tool.driver.rules ?? []) {
			if (rule.id == result.ruleId) {
				return rule?.defaultConfiguration?.level ?? "none";
			}
		}
	}
</script>

<main class="w-full min-h-screen bg-slate-50 text-slate-900 dark:bg-slate-900 dark:text-slate-50">
	<div class="container mx-auto p-6">
		<!-- Header -->
		<div class="mb-8">
			<h1 class="text-3xl font-bold mb-2">SARIF Report Viewer</h1>
			<p class="text-slate-600 dark:text-slate-400">Upload and analyze SARIF reports</p>
		</div>

		<!-- Upload Section -->
		<div class="mb-8">
			<label for="sarif-upload" class="block w-full cursor-pointer">
				<span class="border-2 border-dashed border-slate-300 dark:border-slate-700 rounded-lg p-8 text-center hover:border-slate-400 dark:hover:border-slate-600 transition-colors block">
					<Upload class="mx-auto h-12 w-12 text-slate-400" />
					<p class="mt-4 text-sm text-slate-600 dark:text-slate-400">Click to upload SARIF file or drag and drop</p>
					<p class="mt-2 text-xs text-slate-500">.sarif or .json files</p>
				</span>
				<input id="sarif-upload" type="file" accept=".json,.sarif" onchange={handleFileUpload} class="hidden" />
			</label>

			<!-- Error Message -->
			{#if error}
				<div class="mt-4 p-4 bg-red-100 dark:bg-red-900/20 text-red-700 dark:text-red-400 rounded-lg">
					{error}
				</div>
			{/if}

			<!-- Loading State -->
			{#if isLoading}
				<div class="mt-4 text-center text-slate-600 dark:text-slate-400">Processing SARIF file...</div>
			{/if}
		</div>

		<!-- Results list -->
		{#if results.length > 0}
			<div class="space-y-4">
				{#each results as result}
					{@const Icon = getSeverityIcon(getLevel(result))}
					<div class="bg-white dark:bg-slate-800 rounded-lg shadow-md p-6">
						<div class="flex items-start gap-4">
							<div class="flex-shrink-0">
								<Icon class={getSeverityColor(getLevel(result))} size={24}></Icon>
							</div>
							<div class="flex-1 space-y-4">
								<div>
									<div class="flex items-center gap-2">
										<span class="font-semibold">{result.ruleId}</span>
										<span class={`px-2 py-1 rounded-full text-xs font-medium ${getSeverityColor(getLevel(result))} bg-opacity-10`}>
											{getLevel(result)}
										</span>
									</div>
									<p class="mt-1 text-slate-600 dark:text-slate-300">{result.message.text}</p>
								</div>

								<!-- Main Location -->
								{#each result.locations ?? [] as location}
									<div class="space-y-2" transition:slide={{ duration: 200 }}>
										<div class="text-sm text-slate-500 dark:text-slate-400">
											File: {location?.physicalLocation?.artifactLocation?.uri}
										</div>
										<CodeSnippet code={location?.physicalLocation?.region?.snippet?.text} line={location?.physicalLocation?.region?.startLine ?? 0} />
									</div>
								{/each}

								<!-- Code Flows -->
								{#if result.codeFlows?.length}
									<div class="mt-4 border-t border-slate-200 dark:border-slate-700 pt-4">
										<h4 class="text-sm font-semibold mb-3">Code Flow</h4>
										{#each result.codeFlows[0].threadFlows[0].locations as { location }, index}
											<div class="pl-4 border-l-2 border-slate-300 dark:border-slate-700 mb-4 last:mb-0">
												<div class="flex items-center gap-2 mb-2">
													<span class="inline-flex items-center justify-center w-5 h-5 rounded-full bg-slate-200 dark:bg-slate-700 text-xs font-medium">
														{index + 1}
													</span>
													{#if location?.message}
														<span class="text-sm text-slate-600 dark:text-slate-400">{location.message.text}</span>
													{/if}
												</div>
												<CodeSnippet code={location?.physicalLocation?.region?.snippet?.text} line={location?.physicalLocation?.region?.startLine} />
											</div>
										{/each}
									</div>
								{/if}
							</div>
						</div>
					</div>
				{/each}
			</div>
		{:else if !isLoading}
			<div class="text-center text-slate-600 dark:text-slate-400 py-12" transition:fade>No results to display. Upload a SARIF file to begin.</div>
		{/if}
	</div>
</main>
