<script>
	import { slide } from "svelte/transition";
	import ClosedIcon from "./ClosedIcon.svelte";
	import OpenedIcon from "./OpenedIcon.svelte";
	export let node;
	export let level = 0;

	export let connectedWallet;

	const getCSSClassBackgroundColor = (child) => {
		return connectedWallet === child
			? "backgroundColorHighlight"
			: "backgroundColorBase";
	};

	function toggle() {
		node.expanded = !node.expanded;
	}
</script>


<!-- {JSON.stringify(node)} -->
<li
	on:click={toggle}
	style="padding-left:{level * 1}rem"
	transition:slide
	class={getCSSClassBackgroundColor(node.host)}
>
	{#if !node.expanded}
		<ClosedIcon />
	{:else}
		<OpenedIcon />
	{/if}
	{node.host}
</li>
{#if node.expanded && node.invitees}
	{#each node.invitees as child}
			<svelte:self node={child} level={level + 1} connectedWallet={connectedWallet} />
	{/each}
{/if}

<style>
	li {
		border-bottom: solid 1px #eee;
		margin: 0 0;
		padding: 1rem;
		/* background: #fafafa; */
		/* background: blue; */
		display: flex;
	}

	.backgroundColorBase {
		background: #efdcb3;
	}
	.backgroundColorHighlight {
		background: darkgreen;
	}
</style>
