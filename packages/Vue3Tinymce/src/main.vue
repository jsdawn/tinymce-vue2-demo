<template>
  <div :id="id" class="tiny-textarea"></div>
</template>

<script>
import {
  uuid,
  getTinymce,
  getContent,
  setContent,
  resetContent,
  setModeDisabled,
  getContentStyle,
  imageUploadHandler,
} from '../utils';

import { scriptLoader } from '../scriptLoader';

export default {
  name: 'Vue3Tinymce',
  props: {
    modelValue: String,
    setting: { type: Object, default: () => ({}) },
    setup: Function,
    disabled: Boolean,
    scriptSrc: String,
    debug: Boolean,
  },
  emits: ['update:modelValue', 'init', 'change'],

  data() {
    return {
      mounting: false,
      editor: null,
      id: uuid('tinymce'),
      err: '',
    };
  },

  watch: {
    modelValue(val, oldVal) {
      if (!this.editor || !this.editor.initialized) return;
      if (oldVal === val || val === getContent(this.editor)) return;

      this.printLog({ type: 'propsChanged to setContent' }, val, oldVal);

      if (val === null) return resetContent('', this.editor);
      setContent(this.getModelValue(), this.editor);
    },

    disabled(val) {
      if (!this.editor || !this.editor.initialized) return;
      setModeDisabled(this.editor, val);
    },
  },

  methods: {
    getModelValue() {
      return String(this.modelValue ?? '');
    },

    updateValue(val) {
      this.$emit('update:modelValue', val);
    },

    printLog(e, val, oldVal) {
      if (!this.debug) return;
      console.log(`来自：${e.type} | \n ${val} \n ${oldVal || '--'}`);
    },

    onChanged(e, editor) {
      if (!editor) editor = this.editor;

      const content = getContent(editor);
      this.printLog(e, content);

      this.updateValue(content);
      this.$emit('change', content);
    },

    onInited(editor) {
      setContent(this.getModelValue(), editor);

      if (this.disabled && editor.mode.get() !== 'readonly') {
        setModeDisabled(editor);
      }

      // change input undo redo keyup
      editor.on('change input undo redo', (e) => {
        this.onChanged(e, editor);
      });

      this.$emit('init', editor);
    },

    initEditor() {
      if (getTinymce() === null) {
        this.err = 'tinymce is null';
        return;
      }

      if (this.debug) {
        console.warn('vue3-tinymce 进入debug模式');
      }

      let setting = {
        ...this.setting,
        selector: `#${this.id}`,
        content_style: getContentStyle(this.setting?.content_style),
        setup: (editor) => {
          this.editor = editor;
          if (this.setup) this.setup(editor);
          editor.on('init', () => this.onInited(editor));
        },
      };

      // custom_images_upload true
      if (this.setting.custom_images_upload) {
        setting.images_upload_handler = (...arg) => {
          return imageUploadHandler.apply(null, [this.setting, ...arg]);
        };
      }

      getTinymce().init(setting);
      this.mounting = false;
    },
  },

  mounted() {
    if (getTinymce() !== null) {
      this.initEditor();
      return;
    }

    const scriptSrc =
      this.scriptSrc ??
      'https://cdn.bootcdn.net/ajax/libs/tinymce/6.1.2/tinymce.min.js';

    scriptLoader.load(scriptSrc, this.initEditor);
  },

  activated() {
    if (!this.mounting) this.initEditor();
  },

  deactivated() {
    if (!this.editor) return;
    getTinymce()?.remove(`#${this.id}`);
  },

  beforeUnmount() {
    if (!this.editor) return;
    getTinymce()?.remove(`#${this.id}`);
  },
};
</script>

<style lang=""></style>
