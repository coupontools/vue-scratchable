# 🖼️ vue-scratchable🧽
A Vue.js wrapper component that turns everything into fun scratch cards. Includes touch support without additional dependencies.

It can also calculate percentage value of the scratchables's cleared area.

## 🎉 Installation

Install it with npm:

```
npm i https://github.com/coupontools/vue-scratchable.git
```

## ✨ Usage

Register the component globally:

```js
import VueScratchable from 'vue-scratchable';

Vue.component('vue-scratchable', VueScratchable);
```

Or use it locally in your Single File Components:

```js
import VueScratchable from 'vue-scratchable';

export default {
  /* ... */
  components: {
    VueScratchable,
  },
  /* ... */
};
```

In both cases you are now able to use it in the `<template>` of a component:

```html
<vue-scratchable>
  <h1>Hello Scratchable World</h1>
</vue-scratchable>
```

## ⚙️ Configuration

### 🎰 Slot

| Slot name | Description |
|-----------|-------------|
| default | The content to be scratched free. |

### 💡 Props

| Property | Type | Description |
|----------|------|-------------|
| brushOptions | Object | Configuration object of the "scratcher". See below for possible options. |
| hideOptions | Object | Configuration object of the scratchable layer. See below for possible options. |
| getPercentageCleared | Boolean | Flag to enable the `percentage-update` event which emits the amount of cleared paint as percentage. |
| percentageStride | Number | A stride used while calculating the cleared percentage to reduce calculation time. |

#### 🖊️ Brush options

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| size | Number | 20 | Defines the [`lineWidth` attribute.](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/lineWidth) |
| shape | String | round | Defines the [`lineJoin` attribute.](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/lineJoin) |

#### 🙈 Hide options

There is currently one type of fill that can be applied to the scratchable area: image. Different types are differentiated by the `type` property:

##### 🖼️ Hide option - Image

| Property | Type | Description |
|----------|------|-------------|
| type | String | If you want it to be a scaled image you should set it to 'image'. |
| src | String | A link to an image. Can be an external Link as well as an imported local asset. |

Example:

```js
const hide = {
  type: 'image',
  src: 'https://mdn.mozillademos.org/files/222/Canvas_createpattern.png',
};
```

### 🎈 Events

| Event name | Parameter type | Description |
|------------|----------------|-------------|
| percentage-update | Number | If the `get-percentage-cleared` flag is set the component will emit this event and pass a number calculated from the percentage amount of the cleared area. |

## ✔️ Caveats

1. Img tags with local assets

When using img tags with local imported assets you have to call the `init` method of the component corresponding to the img tag's `@load` event. The `init` method can be accessed through the component's scoped slot.

Example:

```html
<vue-scratchable v-slot="{ init }">
  <img
    :src="require('./img.jpg')"
    @load="init()"
  >
</vue-scratchable>
```

2. Percentage calculation is very taxing on performance

The cleared area percentage calculation has to take every pixel of the canvas into consideration and analyzes whether they are cleared or not. Since this calculation gets called on every draw step this needs a lot of processing power. Because of that this feature is disabled by default and needs to be explicitly activated.

That's also why the `percentage-stride` property should be set wisely and adjusted to your needs. It defines how many pixels should be skipped while calculating. This obviously also decreases the percentage's value accuracy.

## 🧾 License

[MIT](http://opensource.org/licenses/MIT)
