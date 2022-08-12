<template>
	<div id="app">
		<h1>A beautiful parrot got trapped behind some paper.</h1>
		<h2>Scratch them free!</h2>
		<vue-scratchable
			ref="scratchable"
			v-slot="{ init }"
			:brushOptions="brush"
			:hideOptions="hide"
			getPercentageCleared
			@percentage-update="updatePoints"
		>
			<div class="wrapper">
				<img
					:src="
						require('./assets/mehmet-turgut-kirkgoz-vnayhPykN5Q-unsplash.jpg')
					"
					@load="init()"
				/>
				<h3>{{ subline }}</h3>
			</div>
		</vue-scratchable>
		<p>You scratched {{ percentage }}% free.</p>
		<div>
			<h2>Try some different types!</h2>
			<button @click="hideType = 'color'">🟩 Color</button>
			<button @click="hideType = 'pattern'">🏳️‍🌈 Pattern</button>
			<button @click="hideType = 'image'">🖼️ Image</button>
		</div>
		<pre>Photo by <a href="https://unsplash.com/@tkirkgoz?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Mehmet Turgut Kirkgoz</a> on <a href="https://unsplash.com/t/animals?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></pre>
	</div>
</template>

<script>
import VueScratchable from "./components/vue-scratchable.vue";

export default {
	name: "App",
	components: {
		VueScratchable,
	},
	computed: {
		subline() {
			return this.percentage < 100
				? `🎉 There is still ${
						100 - this.percentage
				}% left for me to be free... 🎉`
				: "💚 Thank you for scratching me free! 💚";
		},
		hide() {
			switch (this.hideType) {
				case "image":
					return this.imageHide;
				case "pattern":
					return this.patternHide;
				default:
					return this.colorHide;
			}
		},
	},
	data() {
		return {
			percentage: 0,
			hideType: "color",
			colorHide: {
				type: "color",
				value: "#FFF00F",
			},
			imageHide: {
				type: "image",
				src: "https://hosting4images.com/clipart/scratch/front/scratchfront11.png",
			},
			patternHide: {
				type: "pattern",
				src: "https://hosting4images.com/clipart/scratch/front/scratchfront11.png",
				repeat: "repeat",
			},
			brush: {
				size: 60,
				shape: "round",
			},
		};
	},
	methods: {
		updatePoints(percentage) {
			this.percentage = percentage;
		},
	},
	watch: {
		hideType: function () {
			this.$refs.scratchable.init();
		},
	},
};
</script>

<style>
body {
	background-color: #333;
	margin: 0;
}

#app {
	font-family: "Open Sans", sans-serif;
	color: white;
	display: flex;
	flex-direction: column;
	align-items: center;
	max-width: 1200px;
	margin: 0 auto;
	margin-top: 50px;
}

.vue-scratchable-wrapper {
	background-color: white;
}

h3 {
	color: #2c3e50;
	text-align: center;
}

a {
	color: #2196f3;
}

button {
	height: 3em;
	width: 7em;
	display: inline;
	margin: 0 0.5em 0.5em 0.5em;
}
</style>
