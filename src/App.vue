<template>
	<div id="app">
		<h1>A beautiful parrot got trapped behind some paper.</h1>
		<h2>Scratch them free!</h2>

		<div class="wrapper">
			<vue-scratchable
				ref="scratchable"
				v-slot="{ init }"
				:brushOptions="brush"
				:hideOptions="hide"
				getPercentageCleared
				@percentage-update="updatePoints"
				@initial-load="onLoad"
			>
				<div class="inner-wrapper">
					<img
						:src="
							require('./assets/mehmet-turgut-kirkgoz-vnayhPykN5Q-unsplash.jpg')
						"
						@load="init()"
						style="height: 200px"
					/>
					<h3>{{ subline }}</h3>
				</div>
			</vue-scratchable>
		</div>
		<p>You scratched {{ percentage }}% free.</p>
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
						// eslint-disable-next-line no-mixed-spaces-and-tabs
				  }% left for me to be free... 🎉`
				: "💚 Thank you for scratching me free! 💚";
		},
	},
	data() {
		return {
			percentage: 0,
			hide: {
				type: "image",
				src: "https://digicpn.com/scratch/templates/scratch1.png",
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
		onLoad() {
			console.log("loaded");
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

.wrapper {
	width: 90%;
	max-width: 400px;
}
.inner-wrapper {
	background-color: white;
	width: 100%;
	height: 100%;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	text-align: center;
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
