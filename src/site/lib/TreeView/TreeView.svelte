<script lang="ts">
	import { onMount } from "svelte";
	import { page } from "$app/stores";

	import { ListItem, TextBlock } from "$lib";

	export let tree = [];

	let treeViewState: any;
	let depth = 0;

	onMount(() => {
		// Check localStorage for an existing treeViewState
		// If none exists, use a blank object string
		treeViewState = JSON.parse(localStorage.getItem("treeViewState") ?? "{}");
	});

	// Utility function for converting regular names to kebab case
	const id = (s: string) => s.toLowerCase().split(" ").join("-");

	// Function for expanding/collapsing docs categories
	function toggleExpansion(event: MouseEvent, name: string) {
		event.stopPropagation();

		// Modify treeViewState to have the opposite of the previous entry for the category
		treeViewState[id(name)] = !treeViewState[id(name)];

		// Update value in localStorage for persistence
		localStorage.setItem("treeViewState", JSON.stringify(treeViewState));
	}
</script>

<div class="tree-view">
	{#each tree as { name, path, type, pages, icon }}
		{#if type === "category"}
			{#if depth > 0}
				<div class="subtree" class:expanded={treeViewState?.[id(name)]}>
					<ListItem
						type="expander"
						expanded={treeViewState?.[id(name)]}
						on:click={e => toggleExpansion(e, name)}
					>
						<svelte:fragment slot="icon">
							{@html icon || ""}
						</svelte:fragment>
						{name}
					</ListItem>
					{#if treeViewState?.[id(name)]}
						<div class="subtree-items">
							<svelte:self depth={depth + 1} tree={pages} />
						</div>
					{/if}
				</div>
			{:else}
				<TextBlock class="category-header" variant="bodyStrong">{name}</TextBlock>
                <svelte:self depth={depth + 1} tree={pages} />
			{/if}
		{:else}
			<ListItem
				on:click
				type="navigation"
				selected={`/docs${path}` === $page.url.pathname}
				href="/docs{path}"
			>
				<svelte:fragment slot="icon">
					{@html icon || ""}
				</svelte:fragment>
				{name}
			</ListItem>
		{/if}
	{/each}
</div>

<style lang="scss">
	// Add padding to subtrees for the nesting effect
	.subtree-items {
		padding-inline-start: 24px;
	}

	.tree-view :global(.category-header) {
		inline-size: 100%;
		padding: 10px 16px;
	}
</style>