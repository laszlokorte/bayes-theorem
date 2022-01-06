<script>
	export let probTree
	export let direction
	export let onchange
	export let setProbabilities

	const formatter = new Intl.NumberFormat(navigator.locale, { maximumFractionDigits: 2, minimumFractionDigits: 2 });


	let dragTarget = null
	let dragTrueChild = null
	let dragFalseChild = null
	let dragRef = null

	function dragStart(evt) {
		evt.preventDefault()
		dragTarget = evt.currentTarget;
		
		dragTrueChild = dragTarget.getAttribute('data-child') === 'true'
		dragFalseChild = dragTarget.getAttribute('data-child') === 'false'
	}
	
	function dragMove(evt) {
		const factor = 1000 / 1400;
		if(dragTarget) {
			const dragVertical = (direction === 'vertical')
			const children = dragTrueChild || dragFalseChild
			const percent = dragVertical^children ? (evt.pageX - (dragRef.offsetLeft + dragRef.offsetWidth*(1-factor)/2))/(dragRef.offsetWidth*factor) :  (evt.pageY - (dragRef.offsetTop + dragRef.offsetWidth*(1-factor)/2))/(dragRef.offsetHeight*factor)

			const percentClamped = Math.max(0, Math.min(percent, 1))
			
			const tree = JSON.parse(dragRef.dataset.tree)

			if(dragVertical) {
				if(dragTrueChild) {
					tree.children[0] = percentClamped
				} else if(dragFalseChild) {
					tree.children[1] = percentClamped
				} else {
					tree.self = percentClamped
				}
			} else {				
				if(dragTrueChild) {
					tree.children[0] = percentClamped
				} else if(dragFalseChild) {
					tree.children[1] = percentClamped
				} else {
					tree.self = percentClamped
				}
			}
			onchange(tree, dragTrueChild)
		}
	}
	
	function dragEnd(evt) {
		if(dragTarget) {
			dragTarget = null;
			dragTrueChild = null;
			dragFalseChild = null;
		}
	}

	function clamp(min, max, v) {
		return Math.max(min, Math.min(max, 1*v))
	}


	function wheel(evt) {
		if(!evt.altKey && !evt.shiftKey) {
			return;
		}

		evt.preventDefault();
		evt.stopPropagation()

		const trueChild = evt.currentTarget.getAttribute('data-child') === 'true'
		const falseChild = evt.currentTarget.getAttribute('data-child') === 'false'
		const d = Math.sign(evt.wheelDeltaY) * 0.05

		if(!(trueChild || falseChild) || (evt.shiftKey && evt.altKey)) {
			setProbabilities(probTree, 'self', clamp(0,1, probTree.self + d))
		} else if(trueChild) {
			setProbabilities(probTree, 0, clamp(0,1,probTree.children[0] + d))
		} else if(falseChild) {
			setProbabilities(probTree, 1, clamp(0,1,probTree.children[1] + d))
		}
	}
</script>

<style>
	.dragging {
		fill:  yellow;
	}

	svg {
		width: 100%; 
		height: auto;
	}
</style>

<svelte:window on:mousemove={dragMove}  on:mouseup={dragEnd} />

<div bind:this={dragRef} data-tree={JSON.stringify(probTree)}>
{#if direction=='vertical'}
<svg viewBox="-200 -200 1400 1400" font-size="50" on:wheel={wheel}>
	


	<rect fill="#faf" x="0" y="0" width={probTree.self*1000} height={probTree.children[0]*1000} />
	<rect fill="#0af" x={probTree.self*1000} y="0" width={(1-probTree.self)*1000} height={(probTree.children[1])*1000} />
	<rect fill="#fa0" x="0" y={(probTree.children[0])*1000} width={probTree.self*1000} height={(1-probTree.children[0])*1000} />
	<rect fill="#0a0" x={probTree.self*1000} y={probTree.children[1]*1000} width={(1-probTree.self)*1000} height={(1-probTree.children[1])*1000} />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="0" y2="-50" x1={probTree.self*250 } y1="-50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.self*750} y1="-50" x2={probTree.self*1000-5} y2="-50" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="-50" x2={probTree.self*1000+5} x1={probTree.self*1000+((1-probTree.self)*250)} y1="-50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1="-50" x1={probTree.self*1000+((1-probTree.self)*750)} x2="1000" y2="-50" stroke-dasharray="10 10" />
	
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="0" y2="1050" x1={400} y1="1050" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1="600" y1="1050" x2={1000} y2="1050" stroke-dasharray="10 10" />



	

	<text  text-anchor="middle" dominant-baseline="middle" x={probTree.self*1000 / 2} y={-50}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text  text-anchor="middle" dominant-baseline="middle" x={500+probTree.self*1000 / 2} y={-50}><tspan>Pr(</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan></text>
	<text  text-anchor="middle" dominant-baseline="middle" x={500} y={1050}>1</text>

	{#if probTree.self > 0.5}

	

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="0" x2="1050" y1={probTree.children[1]*250} x1="1050" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[1]*750} x1="1050" y2={probTree.children[1]*1000-5} x2="1050" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[1]*1000+((1-probTree.children[1])*250)} x1="1050" y2={probTree.children[1]*1000+5} x2="1050" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[1]*1000+((1-probTree.children[1])*750)} x1="1050" y2={1000} x2="1050" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="0" x2="50" y1={probTree.children[0]*250} x1="50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[0]*750} x1="50" y2={probTree.children[0]*1000-5} x2="50" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[0]*1000+((1-probTree.children[0])*250)} x1="50" y2={probTree.children[0]*1000+5} x2="50" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[0]*1000+((1-probTree.children[0])*750)} x1="50" y2={1000} x2="50" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="0" x2="-50" y1={350} x1="-50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1="650" x1="-50" y2={1000} x2="-50" stroke-dasharray="10 10" />
	

	<text  text-anchor="start" dominant-baseline="middle" x={1020} y={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="start" dominant-baseline="middle" x={1020} y={500 + probTree.children[1]*500}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text clip-path={null && `path('M0,0L${probTree.self*1000},0L${probTree.self*1000},${probTree.children[0]*1000}L0,${probTree.children[0]*1000}Z') view-box`} text-anchor="start" dominant-baseline="middle" x={20} y={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text clip-path={null && `path('M0,${probTree.children[0]*1000}L${probTree.self*1000},${probTree.children[0]*1000}L${probTree.self*1000},1000L0,1000Z') view-box`} text-anchor="start" dominant-baseline="middle" x={20} y={500 + probTree.children[0]*500}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>

	<text text-anchor="middle" dominant-baseline="middle" x={-50} y={500}>1</text>
	{:else}


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="0" x2="1050" y1={350} x1="1050" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1="650" x1="1050" y2={1000} x2="1050" stroke-dasharray="10 10" />



	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="0" x2="950" y1={probTree.children[1]*250} x1="950" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[1]*750} x1="950" y2={probTree.children[1]*1000-5} x2="950" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[1]*1000+((1-probTree.children[1])*250)} x1="950" y2={probTree.children[1]*1000+5} x2="950" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[1]*1000+((1-probTree.children[1])*750)} x1="950" y2={1000} x2="950" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="0" x2="-50" y1={probTree.children[0]*250} x1="-50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[0]*750} x1="-50" y2={probTree.children[0]*1000-5} x2="-50" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[0]*1000+((1-probTree.children[0])*250)} x1="-50" y2={probTree.children[0]*1000+5} x2="-50" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.children[0]*1000+((1-probTree.children[0])*750)} x1="-50" y2={1000} x2="-50" stroke-dasharray="10 10" />

	<text clip-path={null && `path('M${probTree.self*1000},0L1000,0L1000,${probTree.children[1]*1000}L${probTree.self*1000},${probTree.children[1]*1000}Z') view-box`}  text-anchor="end" dominant-baseline="middle" x={980} y={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text clip-path={null && `path('M${probTree.self*1000},${probTree.children[1]*1000}L1000,${probTree.children[1]*1000}L1000,1000L${probTree.self*1000},1000Z') view-box`} text-anchor="end" dominant-baseline="middle" x={980} y={500 + probTree.children[1]*500}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="end" dominant-baseline="middle" x={-20} y={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="end" dominant-baseline="middle" x={-20} y={500 + probTree.children[0]*500}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="middle" dominant-baseline="middle" x={1050} y={500}>1</text>
	{/if}

	<text clip-path={null && `path('M0,0L${probTree.self*1000},0L${probTree.self*1000},${probTree.children[0]*1000}L0,${probTree.children[0]*1000}Z') view-box`} stroke="#faf" stroke-width="10"  paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={probTree.self*1000 / 2 + (probTree.self > 0.5 ? 50 : 0)} y={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text stroke="#0af" stroke-width="10" clip-path={null && `path('M${probTree.self*1000},0L1000,0L1000,${probTree.children[1]*1000}L${probTree.self*1000},${probTree.children[1]*1000}Z') view-box`} paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={500+probTree.self*1000 / 2 + (probTree.self > 0.5 ? 0 : -50)} y={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text stroke="#fa0" stroke-width="10" clip-path={null && `path('M0,${probTree.children[0]*1000}L${probTree.self*1000},${probTree.children[0]*1000}L${probTree.self*1000},1000L0,1000Z') view-box`} paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={probTree.self*1000 / 2 + (probTree.self > 0.5 ? 50 : 0)} y={500 + probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text stroke="#0a0" stroke-width="10" clip-path={null && `path('M${probTree.self*1000},${probTree.children[1]*1000}L1000,${probTree.children[1]*1000}L1000,1000L${probTree.self*1000},1000Z') view-box`} paint-order="stroke" text-anchor="middle" dominant-baseline="middle" x={500+probTree.self*1000 / 2 + (probTree.self > 0.5 ? 0 : -50)} y={500 + probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>


	<rect pointer-events="all" on:wheel={wheel} data-child="true" fill="none" x="0" y="0" width={probTree.self*1000} height={1000} />
	<rect pointer-events="all" on:wheel={wheel} data-child="false" fill="none" x={probTree.self*1000} y="0" width={(1-probTree.self)*1000} height={1000} />

	<rect on:wheel={wheel} style="cursor: row-resize;" stroke-width="10" paint-order="stroke" stroke="#fff" fill="#333" y={probTree.children[0]*(1000-6)} x="0" height="6" width={probTree.self*1000-5} on:mousedown={dragStart} data-child="true" class:dragging={dragTrueChild} />

	<rect on:wheel={wheel} style="cursor: row-resize;" stroke-width="10" paint-order="stroke" stroke="#fff" fill="#333" y={probTree.children[1]*(1000-6)} x={probTree.self*(1000-6)+5} height="6" width={(1-probTree.self)*1000} on:mousedown={dragStart} data-child="false" class:dragging={dragFalseChild} />

	<rect style="cursor: col-resize;" stroke-width="10" paint-order="stroke" stroke="#fff" fill="#333" x={probTree.self*(1000-6)} y="-100" width="6" height="1100" on:mousedown={dragStart} class:dragging={!dragTrueChild && !dragFalseChild && dragTarget != null} />

</svg>
{:else}
<svg viewBox="-200 -200 1400 1400" font-size="50">
	

	<rect fill="#faf" y="0" x="0" height={probTree.self*1000} width={probTree.children[0]*1000} />
	<rect fill="#fa0" y={probTree.self*1000} x="0" height={(1-probTree.self)*1000} width={(probTree.children[1])*1000} />
	<rect fill="#0af" y="0" x={(probTree.children[0])*1000} height={probTree.self*1000} width={(1-probTree.children[0])*1000} />
	<rect fill="#0a0" y={probTree.self*1000} x={probTree.children[1]*1000} height={(1-probTree.self)*1000} width={(1-probTree.children[1])*1000} />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="0" x2="-50" y1={probTree.self*250 } x1="-50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1={probTree.self*750} x1="-50" y2={probTree.self*1000-5} x2="-50" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="-50" y2={probTree.self*1000+5} y1={probTree.self*1000+((1-probTree.self)*250)} x1="-50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1="-50" y1={probTree.self*1000+((1-probTree.self)*750)} y2="1000" x2="-50" stroke-dasharray="10 10" />
	
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y2="0" x2="1050" y1={400} x1="1050" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" y1="600" x1="1050" y2={1000} x2="1050" stroke-dasharray="10 10" />


	

	<text  text-anchor="end" dominant-baseline="middle" y={probTree.self*1000 / 2} x={-25}><tspan>Pr(</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan></text>
	<text  text-anchor="end" dominant-baseline="middle" y={500+probTree.self*1000 / 2} x={-25}><tspan>Pr(</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan></text>
	<text  text-anchor="middle" dominant-baseline="middle" y={500} x={1050}>1</text>

	{#if probTree.self > 0.5}

	

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="0" y2="1050" x1={probTree.children[1]*250} y1="1050" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[1]*750} y1="1050" x2={probTree.children[1]*1000-5} y2="1050" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[1]*1000+((1-probTree.children[1])*250)} y1="1050" x2={probTree.children[1]*1000+5} y2="1050" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[1]*1000+((1-probTree.children[1])*750)} y1="1050" x2={1000} y2="1050" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="0" y2="50" x1={probTree.children[0]*250} y1="50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[0]*750} y1="50" x2={probTree.children[0]*1000-5} y2="50" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[0]*1000+((1-probTree.children[0])*250)} y1="50" x2={probTree.children[0]*1000+5} y2="50" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[0]*1000+((1-probTree.children[0])*750)} y1="50" x2={1000} y2="50" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="0" y2="-50" x1={350} y1="-50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1="650" y1="-50" x2={1000} y2="-50" stroke-dasharray="10 10" />
	

	<text  text-anchor="middle" dominant-baseline="hanging" y={1020} x={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="middle" dominant-baseline="hanging" y={1020} x={500 + probTree.children[1]*500}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text clip-path={null && `path('M0,0L0,${probTree.self*1000}L${probTree.children[0]*1000},${probTree.self*1000}L${probTree.children[0]*1000},0Z') view-box`} text-anchor="middle" dominant-baseline="hanging" y={20} x={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text  clip-path={null && `path('M${probTree.children[0]*1000},0L${probTree.children[0]*1000},${probTree.self*1000}L1000,${probTree.self*1000}L1000,0Z') view-box`} text-anchor="middle" dominant-baseline="hanging" y={20} x={500 + probTree.children[0]*500}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>

	<text text-anchor="middle" dominant-baseline="middle" y={-50} x={500}>1</text>
	{:else}


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="0" y2="1050" x1={350} y1="1050" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1="650" y1="1050" x2={1000} y2="1050" stroke-dasharray="10 10" />



	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="0" y2="950" x1={probTree.children[1]*250} y1="950" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[1]*750} y1="950" x2={probTree.children[1]*1000-5} y2="950" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[1]*1000+((1-probTree.children[1])*250)} y1="950" x2={probTree.children[1]*1000+5} y2="950" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[1]*1000+((1-probTree.children[1])*750)} y1="950" x2={1000} y2="950" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x2="0" y2="-50" x1={probTree.children[0]*250} y1="-50" stroke-dasharray="10 10" />
	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[0]*750} y1="-50" x2={probTree.children[0]*1000-5} y2="-50" stroke-dasharray="10 10" />


	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[0]*1000+((1-probTree.children[0])*250)} y1="-50" x2={probTree.children[0]*1000+5} y2="-50" stroke-dasharray="10 10" />

	<line marker-end="url(#arrowhead)"  stroke="#333" stroke-width="4" x1={probTree.children[0]*1000+((1-probTree.children[0])*750)} y1="-50" x2={1000} y2="-50" stroke-dasharray="10 10" />

	<text  clip-path={null && `path('M0,${probTree.self*1000}L0,1000L${probTree.children[1]*1000},1000L${probTree.children[1]*1000},${probTree.self*1000}Z') view-box`} text-anchor="middle" dominant-baseline="ideographic" y={980} x={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text  clip-path={null && `path('M${probTree.children[1]*1000},${probTree.self*1000}L${probTree.children[1]*1000},1000L1000,1000L1000,${probTree.self*1000}Z') view-box`} text-anchor="middle" dominant-baseline="ideographic" y={980} x={500 + probTree.children[1]*500}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="middle" dominant-baseline="ideographic" y={-20} x={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="middle" dominant-baseline="ideographic" y={-20} x={500 + probTree.children[0]*500}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>|</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text text-anchor="middle" dominant-baseline="middle" y={1050} x={500}>1</text>
	{/if}

	<text clip-path={null && `path('M0,0L0,${probTree.self*1000}L${probTree.children[0]*1000},${probTree.self*1000}L${probTree.children[0]*1000},0Z') view-box`} stroke="#faf" stroke-width="10"  paint-order="stroke" text-anchor="middle" dominant-baseline="middle" y={probTree.self*1000 / 2} x={probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text  clip-path={null && `path('M0,${probTree.self*1000}L0,1000L${probTree.children[1]*1000},1000L${probTree.children[1]*1000},${probTree.self*1000}Z') view-box`}  stroke="#fa0" stroke-width="10" paint-order="stroke" text-anchor="middle" dominant-baseline="middle" y={500+probTree.self*1000 / 2} x={probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>
	<text  clip-path={null && `path('M${probTree.children[0]*1000},0L${probTree.children[0]*1000},${probTree.self*1000}L1000,${probTree.self*1000}L1000,0Z') view-box`}  stroke="#0af" stroke-width="10" paint-order="stroke" text-anchor="middle" dominant-baseline="middle" y={probTree.self*1000 / 2} x={500 + probTree.children[0]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}</tspan><tspan>)</tspan>
	</text>
	<text  clip-path={null && `path('M${probTree.children[1]*1000},${probTree.self*1000}L${probTree.children[1]*1000},1000L1000,1000L1000,${probTree.self*1000}Z') view-box`} stroke="#0a0" stroke-width="10" paint-order="stroke" text-anchor="middle" dominant-baseline="middle" y={500+probTree.self*1000 / 2} x={500 + probTree.children[1]*1000 / 2}>
		<tspan>Pr(</tspan><tspan>{probTree.labels[1]}&#773;</tspan><tspan>∩</tspan><tspan>{probTree.labels[0]}&#773;</tspan><tspan>)</tspan>
	</text>


	<rect pointer-events="all" on:wheel={wheel} data-child="true" fill="none" x="0" y="0" height={probTree.self*1000} width={1000} />
	<rect pointer-events="all" on:wheel={wheel} data-child="false" fill="none" y={probTree.self*1000} x="0" height={(1-probTree.self)*1000} width={1000} />
	
	<rect on:wheel={wheel} style="cursor: col-resize;" stroke-width="10" paint-order="stroke" stroke="#fff" fill="#333" x={probTree.children[0]*(1000-6)} y="0" width="6" height={probTree.self*1000-5} on:mousedown={dragStart}  data-child="true" class:dragging={dragTrueChild}/>

	<rect on:wheel={wheel} style="cursor: col-resize;" stroke-width="10" paint-order="stroke" stroke="#fff" fill="#333" x={probTree.children[1]*(1000-6)} y={probTree.self*(1000-6)+5} width="6" height={(1-probTree.self)*1000} on:mousedown={dragStart}  data-child="false" class:dragging={dragFalseChild}/>

	<rect style="cursor: row-resize;" stroke-width="10" paint-order="stroke" stroke="#fff" fill="#333" y={probTree.self*(1000-6)} x="-100" height="6" width="1100" on:mousedown={dragStart} class:dragging={!dragTrueChild && !dragFalseChild && dragTarget != null}/>

</svg>
{/if}
	
</div>