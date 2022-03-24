<script>
import _ from 'lodash'

export default {
	props: {
		inputList: {
			type: Array,
			required: false,
			default: () => ([])
		},
		delFunc: Function,
		changeFunc: Function,
		refinedFunc: Function
	},
	setup() {
		return {}
	}
}
</script>

<template>
	<div v-if="inputList.length">
		<div v-for="(item, index) in inputList" class="row">
			<div v-for="(str, i) in item[0].split('')" :class="`col state${item[1][i]}`" @click="changeFunc(index, i)">{{ str }}</div>
			<div @click="delFunc(index)" :class="`col round${~~(index/5)%2}`">❌</div>
		</div>
		<button @click="refinedFunc" class="input_button">集計</button>
	</div>
</template>

<style lang="scss" scoped>
:root {
	--hit: rgb(167,210,141);
	--brow: rgb(252, 201, 72);
}
.row {
    display: flex;
    justify-content: center;
    padding-top: 0.5ex;
    padding-bottom: 0.5ex;
	.col {
		width: 13%;
		height: auto;
		aspect-ratio: 1 / 1;
		font-size: 1.3em;
		border-radius: 0;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		background-color: white;
		border: 0.15em solid #e5e7eb;
		border-radius: 0.25rem;
		text-align: center;
		font-weight: bold;
		vertical-align: middle;
		user-select: none;
		margin: 0 0.3ex;
		cursor: pointer;

		&.state0 {
			background-color: #9ca3af;
			border: solid 0.15em #9ca3af;
			color: white;
		}
		&.state1 {
			background-color: #eab308;
			border: solid 0.15em #eab308;
			color: white;
		}
		&.state2 {
			background-color: #22c55e;
			border: solid 0.15em #22c55e;
			color: white;
		}
		&.state3 {
			background-image: url(/images/close-lime.svg);
			background-color: #9ca3af;
			border: solid 0.15em #9ca3af;
			color: white;
		}
		&.state4 {
			background-image: url(/images/consonant-green.svg);
			background-color: #9ca3af;
			border: solid 0.15em #9ca3af;
			color: white;
		}
		&.state5 {
			background-image: url(/images/vowel-green.svg);
			background-color: #9ca3af;
			border: solid 0.15em #9ca3af;
			color: white;
		}
	}
}

.input_button {
	font-size: 1.5em;
	margin: 5px;
	border: solid #555 0px;
	background-color: #e5e7eb;
	border-radius: 0.25rem;
	color: black;
	cursor: pointer;
	padding: 4px 12px 0;
}
</style>
