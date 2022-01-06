<script>
	export let probTree
	export let direction


</script>

{#if direction=='vertical'}
<svg style="width: 50vmin; height: auto;" viewBox="-200 -200 1400 1400" font-size="50">
	
	<rect fill="#eee" x="-100" y="-100" width="1200" height="1200" />

	<rect fill="#faf" x="0" y="0" width={probTree.self*1000} height={probTree.children[0]*1000} />
	<rect fill="#0af" x={probTree.self*1000} y="0" width={(1-probTree.self)*1000} height={(probTree.children[1])*1000} />
	<rect fill="#fa0" x="0" y={(probTree.children[0])*1000} width={probTree.self*1000} height={(1-probTree.children[0])*1000} />
	<rect fill="#0a0" x={probTree.self*1000} y={probTree.children[1]*1000} width={(1-probTree.self)*1000} height={(1-probTree.children[1])*1000} />


	<line stroke="#333" stroke-width="3" x1="0" y1="-50" x2={probTree.self*1000} y2="-50" stroke-dasharray="10 10" />
	<line stroke="#333" stroke-width="3" x1="1000" y1="-50" x2={probTree.self*1000} y2="-50" stroke-dasharray="10 10" />
	<line stroke="#333" stroke-width="3" x1="0" y1="1050" x2={1000} y2="1050" stroke-dasharray="10 10" />


	<rect stroke-width="5" stroke="#eee" fill="#333" y={probTree.children[0]*(1000-10) -5} x="0" height="10" width={probTree.self*1000-5} />

	<rect stroke-width="5" stroke="#eee" fill="#333" y={probTree.children[1]*(1000-10)-5} x={probTree.self*(1000-10)+5} height="10" width={(1-probTree.self)*1000} />

	<rect stroke-width="5" stroke="#eee" fill="#333" x={probTree.self*(1000-10)-5} y="-100" width="10" height="1100" />


	<rect fill="none" stroke-width="10" stroke="#eee" x="-100" y="-100" width="1200" height="1200" />

	<text stroke="#eee" stroke-width="25" paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={probTree.self*1000 / 2} y={-50}><tspan>Pr(</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan></text>
	<text stroke="#eee" stroke-width="25" paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={500+probTree.self*1000 / 2} y={-50}><tspan>Pr(</tspan><tspan text-decoration="overline">{probTree.labels[0]}</tspan><tspan>)</tspan></text>
	<text stroke="#eee" stroke-width="25" paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={500} y={1050}>1</text>

	<text  text-anchor="middle" dominant-baseline="middle" x={probTree.self*1000 / 2} y={-50}><tspan>Pr(</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan></text>
	<text  text-anchor="middle" dominant-baseline="middle" x={500+probTree.self*1000 / 2} y={-50}><tspan>Pr(</tspan><tspan text-decoration="overline">{probTree.labels[0]}</tspan><tspan>)</tspan></text>
	<text  text-anchor="middle" dominant-baseline="middle" x={500} y={1050}>1</text>

	{#if probTree.self > 0.5}

	<line stroke="#333" stroke-width="3" y1="0" x1="1050" y2={probTree.children[1]*1000} x2="1050" stroke-dasharray="10 10" />
	<line stroke="#333" stroke-width="3" y1="1000" x1="1050" y2={probTree.children[1]*1000} x2="1050" stroke-dasharray="10 10" />

	<line stroke="#333" stroke-width="3" y1="0" x1="50" y2={probTree.children[0]*1000} x2="50" stroke-dasharray="10 10" />
	<line stroke="#333" stroke-width="3" y1="1000" x1="50" y2={probTree.children[0]*1000} x2="50" stroke-dasharray="10 10" />

	<line stroke="#333" stroke-width="3" y1="0" x1="-50" y2={1000} x2="-50" stroke-dasharray="10 10" />
	

	<text text-anchor="start" dominant-baseline="middle" x={1020} y={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan text-decoration="overline">{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="start" dominant-baseline="middle" x={1020} y={500 + probTree.children[1]*500}>
		<tspan>Pr(</tspan><tspan text-decoration="overline">{probTree.labels[1]}</tspan><tspan>|</tspan><tspan text-decoration="overline">{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="start" dominant-baseline="middle" x={20} y={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="start" dominant-baseline="middle" x={20} y={500 + probTree.children[0]*500}>
		<tspan>Pr(</tspan><tspan text-decoration="overline">{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="middle" dominant-baseline="middle" x={-50} y={500}>1</text>
	{:else}


	<line stroke="#333" stroke-width="3" y1="0" x1="1050" y2={1000} x2="1050" stroke-dasharray="10 10" />

	<line stroke="#333" stroke-width="3" y1="0" x1="950" y2={probTree.children[1]*1000} x2="950" stroke-dasharray="10 10" />
	<line stroke="#333" stroke-width="3" y1="1000" x1="950" y2={probTree.children[1]*1000} x2="950" stroke-dasharray="10 10" />

	<line stroke="#333" stroke-width="3" y1="0" x1="-50" y2={probTree.children[0]*1000} x2="-50" stroke-dasharray="10 10" />
	<line stroke="#333" stroke-width="3" y1="1000" x1="-50" y2={probTree.children[0]*1000} x2="-50" stroke-dasharray="10 10" />

	<text text-anchor="end" dominant-baseline="middle" x={980} y={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan text-decoration="overline">{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="end" dominant-baseline="middle" x={980} y={500 + probTree.children[1]*500}>
		<tspan>Pr(</tspan><tspan text-decoration="overline">{probTree.labels[1]}</tspan><tspan>|</tspan><tspan text-decoration="overline">{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="end" dominant-baseline="middle" x={-20} y={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="end" dominant-baseline="middle" x={-20} y={500 + probTree.children[0]*500}>
		<tspan>Pr(</tspan><tspan text-decoration="overline">{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="middle" dominant-baseline="middle" x={1050} y={500}>1</text>
	{/if}

	<text clip-path={`path('M0,0L${probTree.self*1000},0L${probTree.self*1000},${probTree.children[0]*1000}L0,${probTree.children[0]*1000}Z')`} stroke="#faf" stroke-width="10"  paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={probTree.self*1000 / 2} y={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text stroke="#0af" stroke-width="10" clip-path={`path('M${probTree.self*1000},0L1000,0L1000,${probTree.children[1]*1000}L${probTree.self*1000},${probTree.children[1]*1000}Z')`} paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={500+probTree.self*1000 / 2} y={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>∩</tspan><tspan text-decoration="overline">{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text stroke="#fa0" stroke-width="10" clip-path={`path('M0,${probTree.children[0]*1000}L${probTree.self*1000},${probTree.children[0]*1000}L${probTree.self*1000},1000L0,1000Z')`} paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={probTree.self*1000 / 2} y={500 + probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan text-decoration="overline">{probTree.labels[1]}</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text stroke="#0a0" stroke-width="10" clip-path={`path('M${probTree.self*1000},${probTree.children[1]*1000}L1000,${probTree.children[1]*1000}L1000,1000L${probTree.self*1000},1000Z')`} paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={500+probTree.self*1000 / 2} y={500 + probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan text-decoration="overline">{probTree.labels[1]}</tspan><tspan>∩</tspan><tspan text-decoration="overline">{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
</svg>
{:else}
<svg style="width: 28em; height: auto;" viewBox="-100 -100 1200 1200">
	
	<rect fill="#eee" y="-100" x="-100" height="1200" width="1200" />

	<rect fill="#faf" y="0" x="0" height={probTree.self*1000} width={probTree.children[0]*1000} />
	<rect fill="#fa0" y={probTree.self*1000} x="0" height={(1-probTree.self)*1000} width={(probTree.children[1])*1000} />

	<rect fill="#0af" y="0" x={(probTree.children[0])*1000} height={probTree.self*1000} width={(1-probTree.children[0])*1000} />
	<rect fill="#0a0" y={probTree.self*1000} x={probTree.children[1]*1000} height={(1-probTree.self)*1000} width={(1-probTree.children[1])*1000} />

 

	<rect stroke-width="5" stroke="#eee" fill="#333" x={probTree.children[0]*(1000-10) -5} y="0" width="10" height={probTree.self*1000-5} />

	<rect stroke-width="5" stroke="#eee" fill="#333" x={probTree.children[1]*(1000-10)-5} y={probTree.self*(1000-10)+5} width="10" height={(1-probTree.self)*1000} />

	<rect stroke-width="5" stroke="#eee" fill="#333" y={probTree.self*(1000-10)-5} x="-100" height="10" width="1200" />


	<rect fill="none" stroke-width="10" stroke="#eee" y="-100" x="-100" height="1200" width="1200" />
</svg>
{/if}