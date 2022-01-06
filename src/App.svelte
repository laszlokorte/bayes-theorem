<script>
	import Chart from './Chart.svelte'

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
</script>

<style>
	.container {
		display:  grid;
		grid-template-columns: repeat(auto-fill, minmax(25em, 1fr));
		gap: 0 3em;
		max-width: 70em;
		justify-content: center;
		align-content: center;
		padding:  0 1em 1em;
		margin: auto;
		box-sizing: border-box;
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


	.negation {
		border-top: 1px solid black;
		line-height: 0.8em;
		display: inline-block;
	}

	.nowrap {
		white-space: nowrap;
	}
</style>

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
Label: <input required type="text" maxlength="1" size="1" value={probTree.labels[0]} on:input={(evt) => {probTree.labels[0] = evt.currentTarget.value || 'A'}} placeholder="A"></label>
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
		
	</div>

	
<div>
	<div class="slider-container" data-tree={JSON.stringify(transposed)}>
		<label style="grid-column: 1 / -1; justify-self: start;">
Label: <input required type="text" maxlength="1" size="1" value={probTree.labels[1]} on:input={(evt) => {probTree.labels[1] = evt.currentTarget.value || 'B'}} placeholder="B"></label>
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
	</div>

	<Chart onchange={(t) => probTree = t} probTree={probTree} direction="vertical" />

		<Chart onchange={(t) => probTree = baysianTranspose(t)} probTree={transposed} direction="horizontal" />

</div>
