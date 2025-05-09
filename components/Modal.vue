<template>
  <span v-show="isShown">
    <div
      class="modal"
      @mousedown.self="hideWithCheck()"
      style="display: flex; justify-content: center"
    >
      <div
        class="modal-dialog"
        :class="dialogClass"
        style="
          display: flex;
          align-items: center;
          margin-left: 0;
          margin-right: 0;
        "
      >
        <div class="modal-content" style="max-height: 100%">
          <div class="modal-header" style="flex-shrink: 0">
            <h4 class="modal-title">
              <slot name="title">{{ action }}</slot>
            </h4>
            <button
              type="button"
              class="close"
              @click="hide"
              aria-label="Close"
              v-show="!keepOpen"
            >
              &times;
            </button>
          </div>
          <div
            class="modal-body"
            style="overflow: auto; -webkit-overflow-scrolling: auto"
            tabindex="-1"
          >
            <slot></slot>
          </div>
          <div class="modal-footer" v-if="buttons">
            <button
              type="button"
              class="btn btn-secondary"
              @click="hideWithCheck"
              v-if="showCancel"
            >
              Cancel
            </button>
            <button
              type="button"
              class="btn"
              :class="buttonClass"
              @click="submit"
              :disabled="shouldBeDisabled()"
            >
              {{ submitText }}
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop show"></div>
  </span>
</template>

<script lang="ts">
  import { Component, Hook, Prop } from '@f-list/vue-ts';
  import Vue from 'vue';
  import { getKey } from '../chat/common';
  import { Keys } from '../keys';

  const dialogStack: Modal[] = [];
  window.addEventListener('keydown', e => {
    if (getKey(e) === Keys.Escape && dialogStack.length > 0)
      dialogStack[dialogStack.length - 1].hideWithCheck();
  });
  window.addEventListener(
    'backbutton',
    e => {
      if (dialogStack.length > 0) {
        e.stopPropagation();
        e.preventDefault();
        dialogStack[dialogStack.length - 1].hide();
      }
    },
    true
  );

  export let isShowing = false;

  @Component
  export default class Modal extends Vue {
    @Prop({ default: '' })
    readonly action!: string;
    @Prop
    readonly dialogClass?: { string: boolean };
    @Prop({ default: true })
    readonly buttons!: boolean;
    @Prop({ default: () => ({ 'btn-primary': true }) })
    readonly buttonClass!: { string: boolean };
    @Prop
    readonly disabled?: boolean;
    @Prop({ default: true })
    readonly showCancel!: boolean;
    @Prop
    readonly buttonText?: string;
    isShown = false;

    keepOpen = false;
    forcedDisabled = false;

    get submitText(): string {
      return this.buttonText !== undefined ? this.buttonText : this.action;
    }

    forceDisabled(disabled: boolean): void {
      this.forcedDisabled = disabled;
    }

    shouldBeDisabled(): boolean {
      return this.disabled || this.forcedDisabled;
    }

    submit(e: Event): void {
      this.$emit('submit', e);
      if (!e.defaultPrevented) this.hideWithCheck();
    }

    show(keepOpen: boolean = false): void {
      this.keepOpen = keepOpen;
      if (this.isShown) {
        this.$emit('reopen');
        return;
      }
      this.isShown = true;
      dialogStack.push(this);
      this.$emit('open');
      isShowing = true;
    }

    hide(): void {
      this.isShown = false;
      this.$emit('close');
      dialogStack.pop();
      if (dialogStack.length === 0) isShowing = false;
    }

    hideWithCheck(): void {
      if (this.keepOpen) return;
      this.hide();
    }

    @Hook('beforeDestroy')
    beforeDestroy(): void {
      if (this.isShown) this.hide();
    }
  }
</script>

<style>
  .flex-modal .modal-body > .form-group {
    margin-left: 0;
    margin-right: 0;
  }

  .modal-body:focus {
    outline: none !important;
  }
</style>
