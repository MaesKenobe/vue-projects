<template>
  <dialog class="dialog__card">
    <div class="dialog__content">
      <div class="dialog__header">
        <button class="dialog__close" @click="closeDialog"></button>
      </div>
      <div class="dialog__body">
        <form class="dialog__form" id="custom_timer" ref="form" @submit.prevent="saveDialog">
          <input type="text" name="title" class="form__item item__title" :titleStatus="titleStatus" placeholder="Title" :value="newTitle ? newTitle : tempTimer.timerTitle" @input="event => newTitle = event.target.value" required>
          <input type="text" name="description" class="form__item item__description" placeholder="Description" :value="newDescription ? newDescription : tempTimer.timerDescription" @input="event => newDescription = event.target.value">
        </form>
      </div>
      <div class="dialog__footer">
        <div class="dialog__actions">
          <button type="submit" form="custom_timer" class="button button--primary action--save">Save</button>
          <button class="button button--primary action--close" @click="closeDialog">Cancel</button>
        </div>
      </div>
    </div>
  </dialog>
</template>

<script>
export default {
  name: 'dialog_card',
  data () {
    return {
      errors: [],
      dataTimer: {},
      newTitle: "",
      newDescription: "",
      titleStatus: "",
    }
  },
  props: {
    targetId: {
      type: String,
    },
    tempTimer: {
      type: Object,
    },
  },
  methods: {
    closeDialog() {
      // Remove temporary values.
      this.newTitle = "";
      this.newDescription = "";
      // Send close signal.
      this.$emit('closeDialog');
    },
    saveDialog() {
      // Parse data on submit.
      this.newTitle = this.$refs.form.title.value;
      this.newDescription = this.$refs.form.description.value;

      let tempTimer = {
        "timerId": this.targetId,
        "timerTitle": this.newTitle,
        "timerDescription": this.newDescription ? this.newDescription : "",
        "timerStatus": this.tempTimer.timerStatus ? this.tempTimer.timerStatus : "initial",
        "timerBuffer": this.tempTimer.timerBuffer ? this.tempTimer.timerBuffer : 0,
        "timerTimestamp": this.tempTimer.timerTimestamp ? this.tempTimer.timerTimestamp : 0,
      }

      this.$emit('saveDialog', tempTimer);
      this.closeDialog();
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  @import '~@/assets/scss/partials/utilities';

  // Styles.
  .dialog__card {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: -1;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    padding: 0;
    background-color: rgba($black, 0.8);
    visibility: hidden;

    &[open] {
      visibility: visible;
      z-index: 0;

      .dialog__content {
        transform: scale(1);
      }
    }

    .dialog__content {
      position: relative;
      z-index: 1;
      display: flex;
      flex-direction: column;
      width: 100%;
      min-width: 300px;
      min-height: 300px;
      padding: 30px;
      max-width: 95%;
      max-height: 95%;
      border: 1px solid $black;
      border-radius: 10px;
      background-color: $white;
      transform: scale(0);
      transition-delay: transform 10s;

      @include breakpoint(md) {
        width: 600px;
        min-width: 600px;
        min-height: 600px;
      }
    }

    .dialog__header {
      display: flex;
      align-items: center;
      justify-content: flex-end;
      flex-shrink: 0;
      width: 100%;
      padding-bottom: 30px;

      .dialog__close {
        width: 40px;
        height: 40px;
        background-image: url('~@/assets/images/icons/grey-close.svg');
        background-position: center center;
        background-size: contain;
        background-repeat: no-repeat;
        filter: $caribbean-green-hue;
        transition: transform $transition-short;

        &:hover,
        &:active {
          transform: scale(1.2);
        }
      }
    }

    .dialog__body {
      display: flex;
      flex-direction: column;
      flex-grow: 1;
    }
    .dialog__form {
      display: flex;
      flex-direction: column;
      flex-grow: 1;

      .form__item {
        margin-bottom: 1rem;
      }
    }

    .dialog__footer {
      display: flex;
      flex-direction: column;
      flex-shrink: 0;

      .dialog__actions {
        display: flex;
        align-items: center;
        justify-content: flex-end;
        gap: 15px;
      }
    }
  }
</style>
