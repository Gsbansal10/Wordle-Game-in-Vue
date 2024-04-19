<template>
	<div class="word">
		<span v-for="(block, idx) in word" :key="block + idx" :class="block.class">
			{{ block.letter }}
		</span>
	</div>
</template>

<script>
export default {
	props: ["word", "color"],
	data() {
		return {};
	},
};
</script>

<style scoped>
.word {
	margin: 0.5rem;
}

.green {
	background-color: rgba(102, 185, 102, 0.785);
	transition: background-color 0.5s;
}
.gold {
	background-color: rgba(255, 217, 0, 0.649);
	transition: background-color 0.5s;
}
.grey {
	background-color: lightgray;
}
span {
	display: inline-grid;
	/* background: hwb(50 15% 10% / 0.865); */
	/* background: rgb(212, 211, 211); */
	border: 2px solid #ddd;
	font-size: 1.1rem;
	width: 3rem;
	aspect-ratio: 1;
	font-weight: 500;
	place-content: center;
	text-transform: uppercase;
	border-radius: 5px;
	margin: 0 0.25rem;
	vertical-align: top;
}
</style>
