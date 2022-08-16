<template>
	<div class="vue-scratchable">
		<canvas
			class="vue-scratchable__cover"
			ref="canvas"
			@mousedown="mouseDown"
			@mousemove="mouseMove"
			@touchstart="touchDown"
			@touchmove="touchMove"
		/>
		<div class="vue-scratchable__slot-wrapper">
			<slot :init="init"></slot>
		</div>
	</div>
</template>

<script>
import debounce from "lodash/debounce";

export default {
	name: "VueScratchable",
	events: ["percentage-update"],
	props: {
		brushOptions: {
			type: Object,
			default: () => ({
				size: 20,
				shape: "round",
			}),
		},
		hideOptions: {
			type: Object,
			default: () => ({
				type: "image",
				value: "https://hosting4images.com/clipart/scratch/front/scratchfront11.png",
			}),
		},
		getPercentageCleared: {
			type: Boolean,
			default: false,
		},
		percentageStride: {
			type: Number,
			default: 150,
		},
	},
	data() {
		return {
			canvas: null,
			context: null,
			isPressed: false,
			offset: {
				top: 0,
				left: 0,
			},
			position: {
				currX: 0,
				currY: 0,
				lastX: 0,
				lastY: 0,
			},
			observer: null,
			initFlag: false,
		};
	},
	mounted() {
		this.canvas = this.$refs.canvas;
		const debounceInit = debounce(() => this.init(), 200);

		this.observer = new MutationObserver((mutations) => {
			mutations.forEach(({ attributeName }) => {
				if (this.initFlag || !attributeName) return;
				debounceInit();
			});
		});
		this.observer.observe(this.$el, {
			childList: true,
			attributes: true,
			characterData: true,
			subtree: true,
		});

		window.addEventListener("mouseup", this.mouseUp, false);
		window.addEventListener("touchend", this.touchUp, false);
		window.addEventListener("touchcancel", this.touchUp, false);

		this.init();

		window.addEventListener("resize", debounce(this.init, 500));
	},
	destroyed() {
		window.removeEventListener("mouseup", this.mouseUp);
		window.removeEventListener("touchend", this.touchUp);
		window.removeEventListener("touchcancel", this.touchUp);

		window.removeEventListener("resize", debounce(this.init, 500));

		this.observer.disconnect();
	},
	methods: {
		init() {
			this.initFlag = true;
			this.context = this.canvas.getContext("2d");
			this.$nextTick(() => {
				this.fillArea()
					.then(() => {
						this.calculateClearedArea();
						this.initFlag = false;
					})
					.catch((error) =>
						console.error(` Failed to load image!
							Error: ${error.name}
							Message: ${error.message}
						`)
					);
			});
		},

		setCanvasAndContextSize(height, width) {
			this.context.height = height;
			this.context.width = width;
			this.canvas.height = height;
			this.canvas.width = width;
		},

		setOffsets() {
			const { top, left } = this.canvas.getBoundingClientRect();
			this.offset.top = top + document.body.scrollTop;
			this.offset.left = left + document.body.scrollLeft;
		},

		async fillArea() {
			const { src = "" } = this.hideOptions;
			this.context.globalCompositeOperation = "source-over";

			return new Promise((resolve, reject) => {
				const img = new Image();
				img.onload = () => {
					// Height and width based on the loaded image
					let { height, width } = img;
					const elementWidth = this.$el.getBoundingClientRect().width;

					// Scale to take up the entire width of the element
					height = Math.ceil(height * (elementWidth / width));
					width = elementWidth;

					this.setCanvasAndContextSize(height, width);
					this.context.drawImage(img, 0, 0, width, height);
					resolve();
				};
				img.onerror = (error) => reject(error);
				img.src = src;
				img.crossOrigin = "anonymous";
			});
		},

		clearArea() {
			const { width, height } = this.context.canvas;
			this.context.globalCompositeOperation = "destination-out";
			this.context.fillRect(0, 0, width, height);
		},

		draw() {
			const { currX, currY, lastX, lastY } = this.position;
			this.context.beginPath();
			this.context.globalCompositeOperation = "destination-out";
			this.context.lineWidth = this.brushOptions.size;
			this.context.lineJoin = this.brushOptions.shape;
			this.context.moveTo(lastX, lastY);
			if (this.isPressed) {
				this.context.lineTo(currX, currY);
			} else {
				this.context.lineTo(currX + 0.01, currY + 0.01);
			}
			this.context.closePath();
			this.context.stroke();
			this.lastPositionHelper(currX, currY);
			this.calculateClearedArea();
		},

		lastPositionHelper(x, y) {
			this.position.lastX = x;
			this.position.lastY = y;
		},
		currentPositionHelper(x, y) {
			this.position.currX = x - this.offset.left;
			this.position.currY = y - this.offset.top;
		},

		mouseDown({ clientX, clientY }) {
			this.setOffsets();
			this.currentPositionHelper(clientX, clientY);
			this.lastPositionHelper(this.position.currX, this.position.currY);
			this.draw();
			this.isPressed = true;
		},
		mouseMove({ clientX, clientY }) {
			if (!this.isPressed) return;
			this.currentPositionHelper(clientX, clientY);
			this.draw();
		},
		mouseUp() {
			this.isPressed = false;
		},

		touchDown({ targetTouches: [{ clientX, clientY }] }) {
			this.setOffsets();
			this.currentPositionHelper(clientX, clientY);
			this.lastPositionHelper(this.position.currX, this.position.currY);
			this.draw();
			this.isPressed = true;
		},
		touchMove(event) {
			event.preventDefault();
			const {
				targetTouches: [{ clientX, clientY }],
			} = event;
			this.currentPositionHelper(clientX, clientY);
			this.draw();
		},
		touchUp() {
			this.isPressed = false;
		},

		calculateClearedArea() {
			if (!this.getPercentageCleared) return;
			const { width, height } = this.context.canvas;
			const {
				data,
				data: { length },
			} = this.context.getImageData(0, 0, width, height);
			const total = length / this.percentageStride;
			let clearedCount = 0;
			for (let i = clearedCount; i < length; i += this.percentageStride) {
				if (parseInt(data[i], 10) === 0) clearedCount += 1;
			}
			this.$emit(
				"percentage-update",
				Math.round((clearedCount / total) * 100)
			);
		},
	},
};
</script>

<style scoped>
.vue-scratchable {
	position: relative;
}

.vue-scratchable > * {
	user-select: none;
}

.vue-scratchable__cover {
	position: relative;
	z-index: 1;
}

.vue-scratchable__slot-wrapper {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	z-index: 0;
	margin-bottom: 6px;
}
</style>
