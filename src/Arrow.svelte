<script>
	export let direction = 'down'	
	let className;
    export { className as class };
</script>

<style>
	svg {
		width: 100%;
		height: 100%;
		color: #555;
		opacity: 0.5;
	}

</style>

{#if direction=='down'}
<svg class={className} width="30" height="30" viewBox="-15 -15 30 30" preserveAspectRatio="xMidYMax slice">
<line marker-end="url(#arrowhead)" stroke-dasharray="4 2" stroke-width="1.5" vector-effect="non-scaling-stroke" x1="0" y1="-15" x2="0" y2="15" stroke="currentColor"></line>
</svg>
{:else if direction=='up'}
<svg class={className} width="30" height="30" viewBox="-15 -15 30 30" preserveAspectRatio="xMidYMin slice">
<line marker-end="url(#arrowhead)" stroke-dasharray="4 2" stroke-width="1.5" vector-effect="non-scaling-stroke" x2="0" y2="-15" x1="0" y1="15" stroke="currentColor"></line>
</svg>
{:else if direction=='left'}
<svg class={className} width="30" height="30" viewBox="-15 -15 30 30" preserveAspectRatio="xMinYMid slice">
<line marker-end="url(#arrowhead)" stroke-dasharray="4 2" stroke-width="1.5" vector-effect="non-scaling-stroke" y2="0" x2="-15" y1="0" x1="15" stroke="currentColor"></line>
</svg>
{:else if direction=='right'}
<svg class={className} width="30" height="30" viewBox="-15 -15 30 30" preserveAspectRatio="xMaxYMid slice">
<line marker-end="url(#arrowhead)" stroke-dasharray="4 2" stroke-width="1.5" vector-effect="non-scaling-stroke" y1="0" x1="-15" y2="0" x2="15" stroke="currentColor"></line>
</svg>
{/if}