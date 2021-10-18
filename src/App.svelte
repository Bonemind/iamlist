<script>
	import "../node_modules/@picocss/pico/css/pico.classless.min.css";
	import { SvelteToast, toast } from '@zerodevx/svelte-toast';

	const DEBOUNCE_TIMER = 300;
	const ACTIONS_PATH = "/iamactions.json";

	// Current filter string
	let filter;

	// Sourcedata is the full list. filteredactions will hold the current results based on the filter
	let sourceData = {};
	let filteredActions = {};

	// Used for filter debounce
	let timer;

	// Simple debounce function. Will break down if used with multiple functions
	const debounce = func => {
		clearTimeout(timer);
		timer = setTimeout(() => {
			func();
		}, DEBOUNCE_TIMER);
	}

	// Load the source iam list
	async function fetchSourceData() {
		const res = await fetch(ACTIONS_PATH);
		sourceData = await res.json();
		filteredActions = sourceData;
	}

	// Filters actions based on input. Case insensitive
	function applyFilter() {
		const filterHolder = {};
		Object.keys(sourceData).forEach(k => {
			const serviceResults = sourceData[k].filter(a => {
				if (a.toLowerCase().includes(filter.toLowerCase())) {
					return a;
				}
			})
			if (serviceResults.length > 0) {
				filterHolder[k] = serviceResults;
			}
		});
		filteredActions = filterHolder;
		console.log(filteredActions);
	}

	// Copy the current list of displayed actions
	function copyActions() {
		const actionList = Object.keys(filteredActions).flatMap(s => {
			return filteredActions[s];
		});
		navigator.clipboard.writeText(actionList.join("\n"));
		toast.push('Actions copied')
	}

	// Copies a single action
	function copyRow(action) {
		navigator.clipboard.writeText(action);
		toast.push(`Copied ${action}`)
	}


	fetchSourceData();
</script>

<SvelteToast />
<main class="container">
	<div>
		<h2>AWS IAM Actions</h2>
		<p>
			List is compiled from <a href="https://github.com/glassechidna/trackiam">trackiam</a>.
			Click a row to copy that action. Click "Copy Actions" to copy the entire (filtered) list.

			No affiliation with AWS.
		</p>
	</div>
	<div>
		<form>
			<!-- Markup example 2: input is after label -->
			<label for="filter">Search (case insensitive)</label>
			<input type="text" id="filter" bind:value={filter} on:keyup="{() => debounce(applyFilter)}" name="filter" placeholder="For example: sqs" required>
		</form>
		<div>
			<button on:click={copyActions}>Copy Actions</button>
		</div>
	</div>
	<table>
		<thead>
			<tr>
				<td>
					Service
				</td>
				<td>
					Action
				</td>
			</tr>
		</thead>
		<tbody>
			{#each [...Object.keys(filteredActions)] as k}
				{#each filteredActions[k] as a}
				<tr>
					<td>
						{k}
					</td>
					<td on:click={() => copyRow(a)}>
						{a}
					</td>
				</tr>
				{/each}
			{/each}
		</tbody>
	</table>
</main>

<style></style>