---
title: 前端水印方案
date: 2024-05-10 00:00:00
image: cover.jpg
categories: ["前端"]
weight: 1 
---
> 最近接到一个后台水印开发的需求，本以为很简单的功能，结果前前后后改了三版，简直汗流浃背了

### 想当然的第一版
水印算是前端常见功能了，如果项目用的是[antd](https://www.antdv.com/components/watermark)或者[element-plus](https://cn.element-plus.org/zh-CN/component/watermark.html)，那么可以直接使用框架提供的组件，大概十分钟就能搞定。

而公司后台用的是八百年前的vue2 + element，想要水印功能只能自己手动加，不过事情到这里还不是很麻烦，因为github上关于水印现成的包一抓一大把，搜了下有[watermarkjs](https://github.com/brianium/watermarkjs)、[watermark-dom](https://github.com/saucxs/watermark-dom)，用起来也不费劲。

但是我脑子一抽，想着加水印嘛，不过就是用canvas添加文字然后渲染成背景图片，自己公司项目还不用封装各种参数，直接写死就ok，简直太简单了！于是说干就干，三下五除二封装了一个watermark.js：
```js
const watermark = {};

const setWatermark = (str, targetDom) => {
  const id = 'watermark';

  if (document.getElementById(id) !== null) {
    document.body.removeChild(document.getElementById(id));
  }

  const can = document.createElement('canvas');
  can.width = 250;
  can.height = 180;

  const cans = can.getContext('2d');
  cans.rotate(-15 * (Math.PI / 180));
  cans.font = 'lighter 16px Arial';
  cans.fillStyle = 'rgba(200, 200, 200, 0.25)';
  cans.textAlign = 'left';

  cans.fillText(str, 10, 80);

  const div = document.createElement('div');
  const imageUrl = can.toDataURL('image/png');
  div.id = id;
  div.style.pointerEvents = 'none';
  div.style.top = targetDom.offsetTop + 'px';
  div.style.left = targetDom.offsetLeft + 'px';
  div.style.position = 'fixed';
  div.style.zIndex = '100000';
  div.style.width = targetDom.clientWidth + 'px';
  div.style.height = targetDom.clientHeight + 'px';
  div.style.backgroundImage =
    'url(' + imageUrl + '),' + 'url(' + imageUrl + ')';
  div.style.backgroundPosition = `125px 90px, 0 0`;
  document.body.appendChild(div);
  return id;
};

watermark.set = (str, targetDom) => {
  if (document.getElementById('watermark')) {
    document.getElementById('watermark').style.display = 'block';
  } else {
    let id = setWatermark(str, targetDom);
    const timer = setInterval(() => {
      if (document.getElementById(id) === null) {
        id = setWatermark(str, targetDom);
      } else {
        clearInterval(timer);
      }
    }, 2000);
    window.onresize = () => {
      setWatermark(str, targetDom);
    };
  }
};

watermark.show = () => {
  if (document.getElementById('watermark')) {
    document.getElementById('watermark').style.display = 'block';
  }
};
watermark.hide = () => {
  document.getElementById('watermark').style.display = 'none';
};
export default watermark;

```
在layout.vue调用：

```js
mounted() {
    this.setWatermark()
}
methods: {
    setWatermark() {
        const dom = document.getElementById('main-container')
        watermark.set(this.user.username, dom)
        // watermark_switch 控制水印是否展示
        this.webConfig.watermark_switch ? watermark.show() : watermark.hide()
    },
}
```

在开关控制页，每次字段更新之后调用：
```js
setWatermark(flag) {
    flag ? watermark.show() : watermark.hide()
},
```
刷新页面，水印成功展示~开关水印测试也没问题，于是当天就火速上线内测了。


### 稀里糊涂的第二版
正式上线几天后，正值五一假期，我收到了值班技术的消息：部分用户反馈系统没有开水印功能，但是还是展示水印了。

于是立马开始排查，但是登录用户后台看一切正常，值班技术也说没办法复现，这下就难办了。

想了半天，决定在初始样式加了一行 `div.style.block = 'none'`，先确保未开启水印的用户不会出现问题，节假日后再仔细排查。


### 痛定思痛的第三版

节假日回来立刻翻出element-plus源码学习watermark组件的实现，发现他们是采用组件的形式实现的：
```vue
<watermark>
  <div class="main-container" id="main-container">
    {/* 主页面代码 */}
  </div>
</watermark>
```

而watermark代码如下（vue2版本）：
```vue
<template>
  <div class="containerRef" ref="parentRef">
    <slot></slot>
  </div>
</template>
<script>
const getPixelRatio = () => window.devicePixelRatio || 1;
const BaseSize = 2;
const FontGap = 3;
const toLowercaseSeparator = (key) =>
  key.replace(/([A-Z])/g, '-$1').toLowerCase();
const getStyleStr = (style) =>
  Object.keys(style)
    .map((key) => `${toLowercaseSeparator(key)}: ${style[key]};`)
    .join(' ');

function reRendering(mutation, watermarkElement) {
  let flag = false;
  // 是否删除水印节点
  if (mutation.removedNodes.length) {
    flag = Array.from(mutation.removedNodes).some(
      (node) => node === watermarkElement
    );
  }
  // 是否修改过水印dom属性值
  if (mutation.type === 'attributes' && mutation.target === watermarkElement) {
    flag = true;
  }

  return flag;
}
export default {
  name: 'Watermark',
  props: {
    show: Boolean,
    zIndex: { type: Number, default: 9 },
    rotate: { type: Number, default: -19 },
    width: { type: [String, Number], default: 120 },
    height: { type: [String, Number], default: 100 },
    image: { type: String, default: '' },
    content: { type: [String, Array], default: '' },
    font: {
      type: Object,
      default: () => ({
        fontSize: 14,
        fontFamily: 'sans-serif',
        fontStyle: 'normal',
        fontWeight: 'normal',
        color: 'rgba(0, 0, 0, 0.10)'
      })
    },
    clockwise: { type: Boolean, default: true },
    opacity: { type: Number, default: 0.5 },
    gap: { type: Array, default: () => [20, 20] },
    offset: { type: Array, default: () => [0, 0] }
  },
  data() {
    return {
      watermarkRef: null,
      stopObservation: false,
      observe: null
    };
  },
  components: {},
  computed: {
    markStyle() {
      const props = this.$props;
      const [gapX, gapY] = props.gap;
      const [offsetX, offsetY] = props.offset;

      const gapXCenter = gapX / 2;
      const gapYCenter = gapY / 2;
      const offsetTop = offsetY || gapYCenter;
      const offsetLeft = offsetX || gapXCenter;

      const markStyle = {
        zIndex: this.zIndex,
        opacity: this.opacity,
        position: 'absolute',
        left: 0,
        top: 0,
        width: '100%',
        height: '100%',
        pointerEvents: 'none',
        backgroundRepeat: 'repeat'
      };

      let positionLeft = offsetLeft - gapXCenter;
      let positionTop = offsetTop - gapYCenter;
      console.log({ positionLeft });
      console.log({ positionTop });
      if (positionLeft > 0) {
        markStyle.left = `${positionLeft}px`;
        markStyle.width = `calc(100% - ${positionLeft}px)`;
        positionLeft = 0;
      }
      if (positionTop > 0) {
        markStyle.top = `${positionTop}px`;
        markStyle.height = `calc(100% - ${positionTop}px)`;
        positionTop = 0;
      }

      markStyle.backgroundPosition = `${positionLeft}px ${positionTop}px`;

      return markStyle;
    }
  },
  watch: {
    show(val) {
      console.log('watermark reload');
      this.stopObservation = true;
      this.show ? this.renderWatermark() : this.destroyWatermark();
      // 延迟执行
      setTimeout(() => {
        this.stopObservation = false;
      });
    }
  },
  methods: {
    onMutate(records) {
      if (this.stopObservation) return;

      records.forEach((mutation) => {
        if (!reRendering(mutation, this.watermarkRef)) return;
        this.destroyWatermark();
        this.renderWatermark();
      });
    },
    useMutationObserver(target, callback, options) {
      const isSupported = typeof MutationObserver !== 'undefined';
      if (!isSupported) return false;
      const observe = new MutationObserver(callback);
      observe.observe(target, options);
      return observe;
    },
    getMarkSize(ctx) {
      const props = this.$props;
      const { fontSize, fontFamily } = props.font;

      let defaultWidth;
      let defaultHeight;
      const content = props.content;
      const image = props.image;
      const width = props.width;
      const height = props.height;

      if (!image && ctx.measureText) {
        ctx.font = `${Number(fontSize)}px ${fontFamily}`;
        const contents = Array.isArray(content) ? content : [content];
        const widths = contents.map((item) => ctx.measureText(item).width);
        defaultWidth = Math.ceil(Math.max(...widths));
        defaultHeight =
          Number(fontSize.value) * contents.length +
          (contents.length - 1) * FontGap;
      }

      return [width ?? defaultWidth, height ?? defaultHeight];
    },
    rotateWatermark(ctx, rotateX, rotateY, rotate) {
      const direction = this.$props.clockwise ? 1 : -1;
      ctx.translate(rotateX, rotateY);
      ctx.rotate((Math.PI / 180) * Number(rotate) * direction);
      ctx.translate(-rotateX, -rotateY);
    },
    renderWatermark() {
      const props = this.$props;
      const [gapX, gapY] = props.gap;

      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      const rotate = props.rotate;

      if (!ctx) return false;
      if (!this.watermarkRef) {
        this.watermarkRef = document.createElement('div');
      }

      const ratio = getPixelRatio();
      const [markWidth, markHeight] = this.getMarkSize(ctx);
      const canvasWidth = (gapX + markWidth) * ratio;
      const canvasHeight = (gapY + markHeight) * ratio;
      const drawX = (gapX * ratio) / 2;
      const drawY = (gapY * ratio) / 2;
      const drawWidth = markWidth * ratio;
      const drawHeight = markHeight * ratio;
      const rotateX = (drawWidth + gapX * ratio) / 2;
      const rotateY = (drawHeight + gapY * ratio) / 2;
      /** 备选绘图参数 */
      const alternateDrawX = drawX + canvasWidth;
      const alternateDrawY = drawY + canvasHeight;
      const alternateRotateX = rotateX + canvasWidth;
      const alternateRotateY = rotateY + canvasHeight;

      canvas.setAttribute('width', `${canvasWidth * BaseSize}px`);
      canvas.setAttribute('height', `${canvasHeight * BaseSize}px`);

      ctx.save();
      this.rotateWatermark(ctx, rotateX, rotateY, rotate);

      this.fillTexts(ctx, drawX, drawY, drawWidth, drawHeight);
      ctx.restore();
      this.rotateWatermark(ctx, alternateRotateX, alternateRotateY, rotate);
      this.fillTexts(
        ctx,
        alternateDrawX,
        alternateDrawY,
        drawWidth,
        drawHeight
      );
      this.appendWatermark(canvas.toDataURL(), markWidth);
    },
    appendWatermark(base64Url, markWidth) {
      if (!this.watermarkRef) return;
      const props = this.$props;
      const [gapX] = props.gap;

      this.stopObservation = true;
      const attrs = getStyleStr({
        ...this.markStyle,
        backgroundImage: `url('${base64Url}')`,
        backgroundSize: `${(gapX + markWidth) * BaseSize}px`
      });
      this.watermarkRef.setAttribute('style', attrs);
      this.$el.append(this.watermarkRef);
      // 延迟执行
      setTimeout(() => {
        this.stopObservation = false;
      });
    },
    fillTexts(ctx, drawX, drawY, drawWidth, drawHeight) {
      const props = this.$props;
      const { fontSize, fontFamily, fontStyle, fontWeight, color } = props.font;

      const ratio = getPixelRatio();
      const content = props.content;
      const mergedFontSize = Number(fontSize) * ratio;
      ctx.font = `${fontStyle} normal ${fontWeight} ${mergedFontSize}px/${drawHeight}px ${fontFamily}`;
      ctx.fillStyle = color;
      ctx.textAlign = 'center';
      ctx.textBaseline = 'top';
      ctx.translate(drawWidth / 2, 0);
      const contents = Array.isArray(content) ? content : [content];
      console.log(contents);
      // eslint-disable-next-line no-unused-expressions
      contents?.forEach((item, index) => {
        console.log(drawX);
        console.log(drawY + index * (mergedFontSize + FontGap * ratio));
        ctx.fillText(
          item ?? '',
          drawX + 20,
          drawY + index * (mergedFontSize + FontGap * ratio) + 40
        );
      });
    },
    destroyWatermark() {
      if (!this.watermarkRef) return;
      this.watermarkRef.remove();
      this.watermarkRef = undefined;
    }
  },
  mounted() {
    if (!this.show) {
      return
    }
    this.renderWatermark();
    this.$nextTick(() => {
      this.observe = this.useMutationObserver(this.$el, this.onMutate, {
        attributes: true,
        childList: true,
        subtree: true
      });
    });
  },
  beforeDestroy() {
    this.destroyWatermark();
    this.observe.disconnect();
    this.observe = null;
  }
};
</script>
```

其中有这么一个api：`MutationObserver`

#### MutationObserver
MutationObserver 是一个JavaScript API，用于监控DOM（文档对象模型）的变化。当观察的DOM元素或其子元素发生变化时，它会触发一个回调函数。这使得你可以在元素添加、删除或修改时执行特定的操作。

在这里主要用来实现水印的防篡改，防止一些懂web的客户打开f12手动display:none或者直接删掉水印dom元素。

`useMutationObserver`方法利用浏览器原生的MutationObserverAPI创建了一个观察者实例,用于监听指定DOM元素(target)的变动。这里传入的target是该组件渲染的根元素this.$el。

`onMutate`是观察者的回调函数,它会在监听到target元素或其子元素的变动时被触发执行。在回调中首先通过`reRendering`函数判断变动是否影响到了水印元素,如果是,则调用`destroyWatermark`销毁当前的水印元素,然后调用`renderWatermark`重新绘制新的水印元素。

`reRendering`函数的作用是检测变动记录(mutation)中是否包含了对水印元素的删除或修改属性的操作,如果是则返回true。

所以通过这一系列代码,当容器元素或其子元素发生变动时,如果影响到了水印元素,就会自动销毁旧的水印并重绘新的水印,确保水印的持续存在。

