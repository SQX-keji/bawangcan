<template>
  <image
    :mode="node.attr.mode"
    :lazy-load="node.attr.lazyLoad"
    :class="node.classStr"
    :style="newStyleStr || node.styleStr"
    :data-src="node.attr.src"
    :src="node.attr.src"
    @tap="wxParseImgTap"
    @load="wxParseImgLoad"
    />
</template>

<script>
export default {
  name: 'wxParseImg',
  data() {
    return {
      newStyleStr: '',
      preview: true,
    };
  },
  props: {
    node: {
      type: Object,
      default() {
        return {};
      },
    },
  },
  methods: {
    wxParseImgTap(e) {
      if (!this.preview) return;
      const { src } = e.currentTarget.dataset;
      if (!src) return;
      let parent = this.$parent;
      while(!parent.preview || typeof parent.preview !== 'function') {// TODO 遍历获取父节点执行方法
      	parent = parent.$parent;
      }
      parent.preview(src, e);
    },
    // 图片视觉宽高计算函数区
    wxParseImgLoad(e) {
      const { src } = e.currentTarget.dataset;
      if (!src) return;
      const { width, height } = e.mp.detail;
      const recal = this.wxAutoImageCal(width, height);
      const { imageheight, imageWidth } = recal;
      const { padding, mode } = this.node.attr;
      const { styleStr } = this.node;
      const imageHeightStyle = mode === 'widthFix' ? '' : `height: ${imageheight - 220}px;`;
      this.newStyleStr = `${styleStr}; ${imageHeightStyle}; width: ${imageWidth}px; padding: 0 ${+padding}px;`;
    },
    // 计算视觉优先的图片宽高
    wxAutoImageCal(originalWidth, originalHeight) {
      // 获取图片的原始长宽
      const { padding } = this.node.attr;
      const windowWidth = this.node.$screen.width - (2 * padding);
      const results = {};

      if (originalWidth < 60 || originalHeight < 60) {
        const { src } = this.node.attr;
				let parent = this.$parent;
				while(!parent.preview || typeof parent.preview !== 'function') {
					parent = parent.$parent;
				}
				parent.removeImageUrl(src);
        this.preview = false;
      }

      // 判断按照那种方式进行缩放
      if (originalWidth > windowWidth) {
        // 在图片width大于手机屏幕width时候
        results.imageWidth = windowWidth;
        results.imageheight = windowWidth * (originalHeight / originalWidth);
      } else {
        // 否则展示原来的数据
        results.imageWidth = originalWidth;
        results.imageheight = originalHeight;
      }

      return results;
    },
  },
};
</script>
