<script>
	import Arrow from './Arrow.svelte'

	function baysianTranspose(probs) {
		const newSelf = 1*probs.children[0] * probs.self + 1*probs.children[1] * (1 - probs.self)
		return {
			labels: [probs.labels[1], probs.labels[0]],
			self: 1*newSelf,
			children: [
				newSelf==0 ? probs.reverse[0] : 1*probs.children[0] * probs.self / newSelf,
				newSelf==1 ? probs.reverse[1] : (1-probs.children[0]) * probs.self / (1 - newSelf),
			],
			reverse: probs.children
		};
	}
	
	$: probTree = {
		labels: ['A','B'],
		self: 0.5,
		children: [0.5, 0.5],
		reverse: [0.5, 0.5],
	}

	function reset() {
		probTree = {
			labels: ['A','B'],
			self: 0.5,
			children: [0.5, 0.5],
			reverse: [0.5, 0.5],
		}
	}
	
	$: transposed = baysianTranspose(probTree)
		
	$: trueAndTrue = probTree.children[0] * probTree.self;
	$: falseAndTrue = probTree.children[1] * (1-probTree.self);
	$: trueAndFalse = (1-probTree.children[0]) * probTree.self;
	$: falseAndFalse = (1-probTree.children[1]) * (1-probTree.self);
	
	$: fractionDigits = 2
	$: formatter = new Intl.NumberFormat(navigator.locale, { maximumFractionDigits: fractionDigits, minimumFractionDigits: fractionDigits });
	
	let dragTarget = null
	let dragVertical = null
	let dragTrueChild = null
	let dragFalseChild = null
	let dragRef = null
	function dragStart(evt) {
		evt.preventDefault()
		dragTarget = evt.currentTarget;
		
		dragVertical = dragTarget.parentNode.classList.contains('v')
		dragTrueChild = dragTarget.parentNode.classList.contains('true-children')
		dragFalseChild = dragTarget.parentNode.classList.contains('false-children')
		dragRef = dragTarget.closest('.grid').querySelector('.body')
	}
	
	function dragMove(evt) {
		if(dragTarget) {
			const children = dragTrueChild || dragFalseChild
			const percent = dragVertical^children ? (evt.pageX - dragRef.offsetLeft)/dragRef.offsetWidth :  (evt.pageY - dragRef.offsetTop)/dragRef.offsetHeight

			const percentClamped = Math.max(0, Math.min(percent, 1))
			
			if(dragVertical) {
				if(dragTrueChild) {
					probTree.children[0] = percentClamped
				} else if(dragFalseChild) {
					probTree.children[1] = percentClamped
				} else {
					probTree.self = percentClamped
				}
			} else {
				const newTransposed = JSON.parse(dragRef.dataset.tree)
				
				if(dragTrueChild) {
					newTransposed.children[0] = percentClamped
				} else if(dragFalseChild) {
					newTransposed.children[1] = percentClamped
				} else {
					newTransposed.self = percentClamped
				}
				
				probTree = baysianTranspose(newTransposed)
			}
		}
	}
	
	function dragEnd(evt) {
		if(dragTarget) {
			dragTarget = null;
			dragVertical = null;
			dragTrueChild = null;
			dragFalseChild = null;
			dragRef = null;
		}
	}
	
	function setTransposed(old, p, value) {
		if(p==='self') {
			old.self = value
		} else if(p===1 || p === 0) {
			old.children[p] = value
		}
		
		probTree = baysianTranspose(old)
	}
	
	function setProbabilities(old, p, value) {
		if(p==='self') {
			old.self = value
		} else if(p===1 || p === 0) {
			old.children[p] = value
		}
		
		probTree = old
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

		const grid = evt.currentTarget.closest('.grid')
		const child = evt.currentTarget.closest('.children')
		const d = Math.sign(evt.wheelDeltaY) * 0.05
		const old = JSON.parse(grid.querySelector('.body').dataset.tree)

		if(grid.classList.contains('v')) {
			if(!child || (evt.shiftKey && evt.altKey)) {
				setProbabilities(old, 'self', clamp(0,1, old.self + d))
			} else if(child.classList.contains('true-children')) {
				setProbabilities(old, 0, clamp(0,1,old.children[0] + d))
			} else if(child.classList.contains('false-children')) {
				setProbabilities(old, 1, clamp(0,1,old.children[1] + d))
			}
		} else if(grid.classList.contains('h')) {
			if(!child || (evt.shiftKey && evt.altKey)) {
				setTransposed(old, 'self', clamp(0,1,old.self + d))
			} else if(child.classList.contains('true-children')) {
				setTransposed(old, 0, clamp(0,1,old.children[0] + d))
			} else if(child.classList.contains('false-children')) {
				setTransposed(old, 1, clamp(0,1,old.children[1] + d))
			}
		}
	}
</script>

<style>
	.container {
		display:  flex;
		gap: 3em;
		justify-content: center;
		margin:  0 1em 1em;
	}
	
	.grid {
		flex-shrink: 0;
		display: grid;
		width: 45vmin;
		height: 45vmin;
		font-size: 1.8vmin;
		aspect-ratio: 1;
		background: #eee;
		gap: 2px;
		border: 1px solid gray;
		padding:  2px;
	}
	
	.grid.v {
		grid-template-columns: [total-start] 
			minmax(2em, max-content)
			[total-end true-head-start true-children-start body-start] 
			minmax(0, var(--split, 50fr))
			[true-head-end splitter-start true-children-end] 
			2px 
			[false-head-start splitter-end false-children-start] 
			minmax(0, var(--split-c, 50fr))
			[false-head-end false-children-end body-end];
		grid-template-rows:  [splitter-start false-head-start true-head-start] minmax(2em, max-content) [false-head-end true-head-end total-start true-children-start false-children-start body-start] 1fr [total-end splitter-end true-children-end false-children-end body-end];
		
	}
	
	.grid.h {
		grid-template-rows: [total-start] 
			minmax(2em, max-content)
			[total-end true-head-start true-children-start body-start] 
			minmax(0, var(--split, 50fr))
			[true-head-end splitter-start true-children-end] 
			2px
			[false-head-start splitter-end false-children-start] 
			minmax(0, var(--split-c, 50fr))
			[false-head-end false-children-end body-end];
		grid-template-columns:  [splitter-start false-head-start true-head-start] minmax(2em, max-content) [false-head-end true-head-end total-start true-children-start false-children-start body-start] 1fr [total-end splitter-end true-children-end false-children-end body-end];
	}

	
	.total {
		grid-area: total;
		display: grid;
		align-content: center;
		justify-content: center;
	}
	
	.total.v {
		border-right: 2px solid black;
		grid-template-rows: [arrow-a-start] 1fr [arrow-a-end] min-content [arrow-b-start] 1fr [arrow-b-end];
		grid-template-columns: [arrow-a-start arrow-b-start] min-content [arrow-a-end arrow-b-end];
		justify-content: center;
		justify-items: center;
		gap: 0.5em;
	}
	
	.total.h {
		border-bottom: 2px solid black;
		grid-template-columns: [arrow-a-start] 1fr [arrow-a-end] min-content [arrow-b-start]1fr[arrow-b-end];
		grid-template-rows: [arrow-a-start arrow-b-start] min-content [arrow-a-end arrow-b-end];
		align-content: center;
		align-items: center;
		gap: 0.5em;
	}
	
	.head {
		white-space: nowrap;
		padding: 0.1em;
	}
	
	.head.v {
		border-bottom: 2px solid black;
		display: grid;
		grid-template-columns: [arrow-a-start] 1fr [arrow-a-end] min-content [arrow-b-start]1fr[arrow-b-end];
		grid-template-rows: [arrow-a-start arrow-b-start] min-content [arrow-a-end arrow-b-end];
		align-content: center;
		align-items: center;
		height: min-content;
		gap: 0.5em;
	}
	
	.head.h {
		border-right: 2px solid black;
		display: grid;
		grid-template-rows: [arrow-a-start] 1fr [arrow-a-end] min-content [arrow-b-start]1fr[arrow-b-end];
		grid-template-columns: [arrow-a-start arrow-b-start] min-content [arrow-a-end arrow-b-end];
		justify-content: center;
		justify-items: center;
		width: min-content;
		gap: 0.5em;
	}
	
	.splitter {
		grid-area: splitter;
		background: #666;
		display: grid;
		align-items: stretch;
		justify-items: stretch;
	}
	
	.splitter.state-dragging {
		border: 2px solid yellow;
	}
	
	.splitter::before {
		content: '';
		margin: -4px;
		display: block;
		z-index: 1;
	}
	
	.true-head {
		grid-area: true-head;
		text-align: center;
		display: grid;
		align-content: center;
	}
	
	.false-head {
		grid-area: false-head;
		text-align: center;
		display: grid;
		align-content: center;
	}
	
	.true-children {
		grid-area: true-children;
	}
	
	.false-children {
		grid-area: false-children;
	}
	
	.children {
		max-width: 100%;
		max-height: 100%;
		overflow: hidden;
	}
	
	.children.v {
		display: grid;
		grid-template-rows: minmax(0, var(--split, 50fr))
			[splitter-start] 
			2px
			[splitter-end] 
			minmax(0, var(--split-c, 50fr));
		grid-template-columns: [splitter-start]  1fr [splitter-end];
		gap: 2px;
	}
	
	.children.h {
		display: grid;
		grid-template-columns: minmax(0, var(--split, 50fr))
			[splitter-start] 
			2px
			[splitter-end] 
			minmax(0, var(--split-c, 50fr));
		grid-template-rows: [splitter-start]  1fr [splitter-end];
		gap: 2px;
	}
	
	.inner {
		background: green;
		overflow: hidden;
		display: grid;
		align-content: center;
		justify-content: center;
		width: 100%;
		height: 100%;
	}
	
	.inner.v {
		grid-template-columns: 1fr [center-start] 10fr [center-end side-start arrow-a-start arrow-b-start] 1fr [side-end arrow-a-end arrow-b-end];
		grid-template-rows: [full-start arrow-a-start center-start] 1fr [side-start arrow-a-end] max-content [side-end arrow-b-start] 1fr [full-end arrow-b-end center-end];
		align-items: center;
		justify-items: center;
		gap: 0.5em;
	}
	
	.inner.h {
		grid-template-rows: 1fr [center-start] 10fr [center-end side-start arrow-a-start arrow-b-start] 1fr [side-end arrow-a-end  arrow-b-end];
		grid-template-columns: [full-start center-start arrow-a-start] 1fr [side-start arrow-a-end] max-content [side-end arrow-b-start] 1fr [side-end full-end center-end arrow-b-end];
		align-items: center;
		justify-items: center;
		gap: 0.5em;
	}
	
	:global(.side-arrow-a) {
		grid-area: arrow-a;
		align-self: stretch;
		justify-self: stretch;
	}
	
	:global(.side-arrow-b) {
		grid-area: arrow-b;
		align-self: stretch;
		justify-self: stretch;
	}
	
	.area {
		grid-area: center;
		text-align: center;
		background: #fff3;
		padding: 0.5em;
		margin: 0.5em;
		border-radius: 0.5em;
	}
	
	.side {
		grid-area: side;
		padding: 0.1em;
		white-space: nowrap;
	}
	
	.children.v > .splitter {
		cursor: row-resize;
	}
	
	.children.h > .splitter {
		cursor: col-resize;
	}
	
	.grid.v > .splitter {
		cursor: col-resize;
	}
	
	.grid.h > .splitter {
		cursor: row-resize;
	}
	
	.inner.tt {
		background: #fbf;		
	}
	
	.inner.tf {
		background: #fb0;		
	}
	
	.inner.ff {
		background: #0b0;		
	}
	
	.inner.ft {
		background: #0bf;		
	}
	
	.body {
		grid-area: body;
		pointer-events: none;
	}
	
	.slider-container {
		display: grid;
		grid-template-columns: max-content 1fr max-content;
		gap: 1em;
		width: 100%;
		align-items: center;
		margin: 1em 0;
	}
	
	input[type=range] {
		width: 100%;
		margin: 0;
	}
	
	input[type=number] {
		width: 5em;
		margin: 0;
	}

	.titel-bar {
		margin: 0;
		padding: 0;
		font-size: 1.2em;
		background: #333;
		color: #fff;
		padding: 0.5em;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	:global(body) {
		margin: 0;
		padding: 0;
	}

	.reset {
		align-self: flex-start;
		cursor: pointer;
	}

	.omega {
		text-align: center;
		transform: rotate(-45deg);
		align-self: center;
		justify-self: center;
		color: #666;
		font-size: 0.9em;
		padding: 4px;
	}

	.negation {
		border-top: 1px solid black;
		line-height: 0.8em;
		display: inline-block;
	}

	.nowrap {
		white-space: nowrap;
	}
</style>

<svelte:window on:mousemove={dragMove}  on:mouseup={dragEnd} />

<div class="titel-bar">
	Dependent probabilities

	<span style="font-size: 1rem; font-weight: normal;">
		Precision: 
		<input type="number" size="3" bind:value={fractionDigits} min="2" max="3">
	</span>
</div>

<p style="text-align: center;">
	To adjust the probabilities use the slider controls or drag the borders of the colored rectangles directly.
</p>

<center>
	<button class="reset" on:click={reset}>Reset</button>
</center>

<div class="container">
	<div>
		<div class="slider-container" data-tree={JSON.stringify(probTree)}>
<label style="grid-column: 1 / -1; justify-self: start;">
Label: <input required type="text" maxlength="3" size="3" value={probTree.labels[0]} on:input={(evt) => {probTree.labels[0] = evt.currentTarget.value || 'A'}} placeholder="A"></label>
<span>Pr({probTree.labels[0]}):</span> 
<input style="accent-color:black" type="range" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(probTree.self)} on:input={(evt) => setProbabilities(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 'self', evt.currentTarget.valueAsNumber)} />
<input length="4" type="number" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(probTree.self)} on:input={(evt) => setProbabilities(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 'self', evt.currentTarget.valueAsNumber)} />
<span>Pr({probTree.labels[1]}|{probTree.labels[0]}):</span> 
<input style="accent-color:#faf" type="range" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(probTree.children[0])} on:input={(evt) => setProbabilities(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 0, evt.currentTarget.valueAsNumber)} />
<input type="number" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(probTree.children[0])} on:input={(evt) => setProbabilities(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 0, evt.currentTarget.valueAsNumber)} />
<span>Pr({probTree.labels[1]}|<span class="negation">{probTree.labels[0]}</span>): </span>
<input style="accent-color:#0af" type="range" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(probTree.children[1])} on:input={(evt) => setProbabilities(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 1, evt.currentTarget.valueAsNumber)} />
<input type="number" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(probTree.children[1])} on:input={(evt) => setProbabilities(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 1, evt.currentTarget.valueAsNumber)} />

</div>
		
<div on:wheel={wheel} class="grid v" style="--split: {Math.round(probTree.self * Math.pow(10, fractionDigits))}fr;--split-c: {Math.pow(10, fractionDigits) - Math.round(probTree.self * Math.pow(10, fractionDigits))}fr;">
	<div class="omega">
		Pr(&Omega;)
	</div>
	<div class="body" data-tree={JSON.stringify(probTree)}>
		
	</div>
	<div class="total v">
		<Arrow direction="up" class="side-arrow-a" />
		<Arrow direction="down" class="side-arrow-b" />
		1
	</div>
	<div class="true-head head v">
		<span><span class="nowrap">Pr({probTree.labels[0]})</span>
		<br>={formatter.format(probTree.self)}</span>

		<Arrow direction="left" class="side-arrow-a" />
		<Arrow direction="right" class="side-arrow-b" />
	</div>
	<div class="false-head head v">
		<span><span class="nowrap">Pr(<span class="negation">{probTree.labels[0]}</span>)</span>
		<br>={formatter.format(1-probTree.self)}</span>

		<Arrow direction="left" class="side-arrow-a" />
		<Arrow direction="right" class="side-arrow-b" />
	</div>
	<div class="splitter" on:mousedown={dragStart} class:state-dragging={dragVertical && !dragTrueChild && !dragFalseChild}></div>
	
	<div on:wheel={wheel} class="true-children children v" style="--split: {Math.round(probTree.children[0] * Math.pow(10, fractionDigits))}fr;--split-c: {Math.pow(10, fractionDigits) - Math.round(probTree.children[0] * Math.pow(10, fractionDigits))}fr;">
		<div class="inner tt v">
			<span class="area"><span class="nowrap">Pr({probTree.labels[1]}∩{probTree.labels[0]})</span>
				<br>={formatter.format(trueAndTrue)}
			</span>

			<Arrow direction="up" class="side-arrow-a" />
			<Arrow direction="down" class="side-arrow-b" />
			<div class="side">
				<span class="nowrap">Pr({probTree.labels[1]}|{probTree.labels[0]})</span>
				<br>={formatter.format(probTree.children[0])}
			</div>
		</div>
		<div class="splitter" on:mousedown={dragStart} class:state-dragging={dragVertical && dragTrueChild && !dragFalseChild}></div>
		<div class="inner tf v">
			<span class="area">
			<span class="nowrap">Pr(<span class="negation">{probTree.labels[1]}</span>∩{probTree.labels[0]})</span><br>={formatter.format(trueAndFalse)}</span>

			<Arrow direction="up" class="side-arrow-a" />
			<Arrow direction="down" class="side-arrow-b" />
			<div class="side">
				<span class="nowrap">Pr(<span class="negation">{probTree.labels[1]}</span>|{probTree.labels[0]})</span>
				<br>={formatter.format(1-probTree.children[0])}
			</div>
		</div>
	</div>
	
	<div on:wheel={wheel} class="false-children children v" style="--split: {Math.round(probTree.children[1] * Math.pow(10, fractionDigits))}fr;--split-c: {Math.pow(10, fractionDigits) - Math.round(probTree.children[1] * Math.pow(10, fractionDigits))}fr;">
		<div class="inner ft v">
			<span class="area">
			<span class="nowrap">Pr({probTree.labels[1]}∩<span class="negation">{probTree.labels[0]}</span>)</span><br>={formatter.format(falseAndTrue)}</span>


			<Arrow direction="up" class="side-arrow-a" />
			<Arrow direction="down" class="side-arrow-b" />
			<div class="side">
				<span class="nowrap">Pr({probTree.labels[1]}|<span class="negation">{probTree.labels[0]}</span>)</span>
				<br>={formatter.format(probTree.children[1])}
			</div></div>
		<div class="splitter" on:mousedown={dragStart} class:state-dragging={dragVertical && !dragTrueChild && dragFalseChild}></div>
		<div class="inner ff v">
			<span class="area">
			<span class="nowrap">Pr(<span class="negation">{probTree.labels[1]}</span>∩<span class="negation">{probTree.labels[0]}</span>)</span><br>={formatter.format(falseAndFalse)}
			</span>

			<Arrow direction="up" class="side-arrow-a" />
			<Arrow direction="down" class="side-arrow-b" />
			<div class="side">
				<span class="nowrap">Pr(<span class="negation">{probTree.labels[1]}</span>|<span class="negation">{probTree.labels[0]}</span>)</span>
				<br>={formatter.format(1-probTree.children[1])}
			</div>
		</div>
	</div>
</div>
	</div>

	
<div>
	<div class="slider-container" data-tree={JSON.stringify(transposed)}>
		<label style="grid-column: 1 / -1; justify-self: start;">
Label: <input required type="text" maxlength="3" size="3" value={probTree.labels[1]} on:input={(evt) => {probTree.labels[1] = evt.currentTarget.value || 'B'}} placeholder="B"></label>
<span>Pr({transposed.labels[0]}): </span>
<input style="accent-color:#000" type="range" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(transposed.self)} on:input={(evt) => setTransposed(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 'self', evt.currentTarget.valueAsNumber)} />
<input length="4" type="number" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(transposed.self)} on:input={(evt) => setTransposed(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 'self', evt.currentTarget.valueAsNumber)} />
<span>Pr({transposed.labels[1]}|{transposed.labels[0]}): </span>
<input style="accent-color:#faf" type="range" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(transposed.children[0])} on:input={(evt) => setTransposed(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 0, evt.currentTarget.valueAsNumber)} />
<input type="number" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(transposed.children[0])} on:input={(evt) => setTransposed(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 0, evt.currentTarget.valueAsNumber)} />
<span>Pr({transposed.labels[1]}|<span class="negation">{transposed.labels[0]}</span>):</span> 
<input style="accent-color:#fa0" type="range" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(transposed.children[1])} on:input={(evt) => setTransposed(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 1, evt.currentTarget.valueAsNumber)} />
<input type="number" min="0" step={Math.pow(10,-fractionDigits)} max="1" value={formatter.format(transposed.children[1])} on:input={(evt) => setTransposed(JSON.parse(evt.currentTarget.parentNode.dataset.tree), 1, evt.currentTarget.valueAsNumber)} />
</div>
<div on:wheel={wheel} class="grid h"  style="--split: {Math.round(transposed.self * Math.pow(10, fractionDigits))}fr;--split-c: {Math.pow(10, fractionDigits) - Math.round(transposed.self * Math.pow(10, fractionDigits))}fr;">
	<div class="omega">
		Pr(&Omega;)
	</div>
	<div class="body" data-tree={JSON.stringify(transposed)}></div>
	<div class="total h">
		<Arrow direction="left" class="side-arrow-a" />
		<Arrow direction="right" class="side-arrow-b" />
		1
	</div>
	<div class="true-head head h">
		<span><span class="nowrap">Pr({transposed.labels[0]})</span>
		<br>={formatter.format(transposed.self)}</span>

		<Arrow direction="up" class="side-arrow-a" />
		<Arrow direction="down" class="side-arrow-b" />
	</div>
	<div class="false-head head h">
		<span><span class="nowrap">Pr(<span class="negation">{transposed.labels[0]}</span>)</span>
		<br>={formatter.format(1-transposed.self)}
		</span>

		<Arrow direction="up" class="side-arrow-a" />
		<Arrow direction="down" class="side-arrow-b" />
	</div>
	<div class="splitter" on:mousedown={dragStart} class:state-dragging={dragVertical===false && !dragTrueChild && !dragFalseChild}></div>
	
	<div on:wheel={wheel} class="true-children children h" style="--split: {Math.round(transposed.children[0] * Math.pow(10, fractionDigits))}fr;--split-c: {Math.pow(10, fractionDigits) - Math.round(transposed.children[0] * Math.pow(10, fractionDigits))}fr;">
		<div class="inner tt h">
			<span class="area">
			<span class="nowrap">Pr({transposed.labels[1]}∩{transposed.labels[0]})</span><br>={formatter.format(trueAndTrue)}
			</span>
			<Arrow direction="left" class="side-arrow-a" />
			<Arrow direction="right" class="side-arrow-b" />
			<div class="side">
				<span class="nowrap">Pr({transposed.labels[1]}|{transposed.labels[0]})</span>
				<br>={formatter.format(transposed.children[0])}
			</div>
		</div>
		<div class="splitter" on:mousedown={dragStart} class:state-dragging={dragVertical===false && dragTrueChild && !dragFalseChild}></div>
		<div class="inner ft h">
			<span class="area">
			<span class="nowrap">Pr(<span class="negation">{transposed.labels[1]}</span>∩{transposed.labels[0]})</span>
			<br>={formatter.format(falseAndTrue)}
			</span>
			<Arrow direction="left" class="side-arrow-a" />
			<Arrow direction="right" class="side-arrow-b" />
			<div class="side">
				<span class="nowrap">Pr(<span class="negation">{transposed.labels[1]}</span>|{transposed.labels[0]})</span>
				<br>={formatter.format(1-transposed.children[0])}
			</div>
		</div>
	</div>
	
	<div on:wheel={wheel} class="false-children children h" style="--split: {Math.round(transposed.children[1] * Math.pow(10, fractionDigits))}fr; --split-c: {Math.pow(10, fractionDigits) - Math.round(transposed.children[1] * Math.pow(10, fractionDigits))}fr;">
		<div class="inner tf h">
			<span class="area">
			<span class="nowrap">Pr({transposed.labels[1]}∩<span class="negation">{transposed.labels[0]}</span>)</span>
			<br>={formatter.format(trueAndFalse)}
			</span>
			<Arrow direction="left" class="side-arrow-a" />
			<Arrow direction="right" class="side-arrow-b" />
			<div class="side">
				<span class="nowrap">Pr({transposed.labels[1]}|<span class="negation">{transposed.labels[0]}</span>)</span>
				<br>={formatter.format(transposed.children[1])}
			</div>
		</div>
		<div class="splitter" on:mousedown={dragStart} class:state-dragging={dragVertical===false && !dragTrueChild && dragFalseChild}></div>
		<div class="inner ff h">
			<span class="area">
			<span class="nowrap">Pr(<span class="negation">{transposed.labels[1]}</span>∩<span class="negation">{transposed.labels[0]}</span>)</span>
			<br>={formatter.format(falseAndFalse)}
			</span>
			<Arrow direction="left" class="side-arrow-a" />
			<Arrow direction="right" class="side-arrow-b" />
			<div class="side">
				<span class="nowrap">Pr(<span class="negation">{transposed.labels[1]}</span>|<span class="negation">{transposed.labels[0]}</span>)</span>
				<br>={formatter.format(1-transposed.children[1])}
			</div>
		</div>
	</div>
</div>
	
	</div>


</div>
