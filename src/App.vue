<template>
  <div class="tiny-page">
    <h1>Vue2 Tinymce</h1>

    <vue3-tinymce modelValue.sync="content" />
  </div>
</template>

<script>
import Vue3Tinymce from '../packages/Vue3Tinymce';

export default {
  components: { Vue3Tinymce },

  data() {
    return {
      content: 'tinymce 默认的编辑器设置，不需要传参额外的setting ...',
    };
  },
};
</script>

<style lang="css">
.tiny-page {
  display: flex;
  flex-direction: column;
}
</style>
