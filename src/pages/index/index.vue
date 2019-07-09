<template>
  <div class="intro">
    <p class="intro-tips">
      请选择难度
    </p>
    <button @click="onSelect('easy')">简单</button>
    <button @click="onSelect('medium')">中等</button>
    <button @click="onSelect('hard')">困难</button>
  </div>
</template>
<script>
export default {
  methods: {
    /**
     * 选择难度
    */
    onSelect(quality) {
      wx.navigateTo({
        url: `../suduko/main?quality=${quality}`,
      });
    },
  },
};
</script>
<style lang="scss" scoped>
.intro {
  text-align: center;
  &-tips {
    color: blue;
  }
}
</style>
