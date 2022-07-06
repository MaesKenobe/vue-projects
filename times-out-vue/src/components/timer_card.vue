<template>
  <div class="timer__card" :id="cardId">
    <div class="top__wrapper">
      <div class="text__box">
        <h2 class="card__title">{{ cardTitle }}</h2>
        <p class="card__description">{{ cardDescription }}</p>
      </div>
      <div class="timer__box">
        <span class="timer__rendered">{{ transformTime(cardBuffer) }}</span>
      </div>
      <div class="actions__box">
        <input type="button" class="button edit__timer" @click="editTimer">
        <input type="button" class="button remove__timer" @click="removeTimer">
      </div>
    </div>
    <div class="controls__box">
      <input v-if="cardStatus === 'initial'" type="button" class="button change__timer initial" value="Start" @click="changeTimerStatus('init')">
      <input v-else-if="cardStatus === 'running'" type="button" class="button change__timer running" value="Pause" @click="changeTimerStatus('pause')">
      <input v-else-if="cardStatus === 'paused'" type="button" class="button change__timer paused" value="Restart" @click="changeTimerStatus('restart')">
    </div>
  </div>
</template>

<script>
export default {
  name: 'timer_card',
  data () {
    return {
      newTimestamp: this.cardTimestamp,
    }
  },
  props: {
    cardId: {
      type: String,
      required: true,
    },
    cardTitle: {
      type: String,
      required: true,
      default: "Title",
    },
    cardDescription: {
      type: String,
      required: false,
      default: "Description",
    },
    cardStatus: {
      type: String,
      required: true,
      default: "initial",
    },
    cardBuffer: {
      type: Number,
      default: 0,
    },
    cardTimestamp: {
      type: Number,
      default: new Date(),
    },
  },
  methods: {
    editTimer() {
      this.$emit('editTimer', this.cardId);
    },
    removeTimer() {
      this.$emit('removeTimer', this.cardId);
    },
    changeTimerStatus(action) {
      let status = "";
      // Define newTimestamp.
      this.newTimestamp = new Date().getTime();

      if (action === 'init') {
        status = 'running';
      }

      if (action === 'pause') {
        status = 'paused';
      }

      if (action === 'restart') {
        status = 'running';
      }

      // Emit signal and parse parameters.
      this.$emit('changeTimerStatus', this.cardId, status, this.newTimestamp);

      // Set variables to default.
      status = "";
    },
    transformTime(seconds) {
      // Just in case the value is not a number (probably another string).
      let sec = Number(seconds);
      let y = Math.floor(sec / 31536000);
      let mo = Math.floor((sec % 31536000) / 2628000);
      let d = Math.floor(((sec % 31536000) % 2628000) / 86400);
      let h = Math.floor((sec % (3600 * 24)) / 3600);
      let m = Math.floor((sec % 3600) / 60);
      let s = Math.floor(sec % 60);

      let yDisplay = y > 0 ? y + 'y ' : "";
      let moDisplay = mo > 0 ? mo + 'm ' : "";
      let dDisplay = d > 0 ? d + 'd ' : "";
      let hDisplay = h > 0 ? ('0' + h.toString()).slice(-2) + ':' : "00:";
      let mDisplay = m > 0 ? ('0' + m.toString()).slice(-2) + ':' : "00:";
      let sDisplay = s > 0 ? ('0' + s.toString()).slice(-2) : "00";
      return yDisplay + moDisplay + dDisplay + hDisplay + mDisplay + sDisplay;
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  @import '~@/assets/scss/partials/utilities';

  // Styles.
  .timer__card {
    display: flex;
    flex-direction: column;
    background-color: $white;
    box-shadow: 0 0 15px 0 rgba($black, 0.2);
    border-radius: 10px;
    overflow: hidden;

    .top__wrapper {
      display: flex;
      flex-direction: column;
      flex-grow: 1;
      width: 100%;
      padding: 15px;
      border-top: 1px solid $silver;
      border-right: 1px solid $silver;
      border-left: 1px solid $silver;
      border-radius: 10px 10px 0 0;
    }

    .text__box {
      flex-grow: 1;
    }
    .controls__box {
      width: 100%;
      overflow: hidden;
    }

    .card__title {
      margin-top: 0;
      margin-bottom: 16px;
    }

    .card__description {
      margin-bottom: 16px;
    }

    .timer__box {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;

      .timer__rendered {
        width: 100%;
        text-align: center;
        @include font-size(24px);
        font-weight: bold;
        margin-bottom: 16px;
      }
    }

    .actions__box {
      display: flex;
      align-items: center;
      justify-content: flex-end;
      gap: 15px;
    }

    .edit__timer,
    .remove__timer {
      width: 20px;
      height: 20px;
      padding: 0;
      background-position: center center;
      background-size: contain;
      background-repeat: no-repeat;
      transition: opacity $transition-short;

      &:hover,
      &:active {
        opacity: 0.4;
      }
    }

    .edit__timer {
      background-image: url('~@/assets/images/icons/icon-edit.svg');
    }

    .remove__timer {
      background-image: url('~@/assets/images/icons/icon-delete.svg');
    }

    .change__timer {
      width: 100%;
      height: 100%;
      padding: 15px;
      @include h3(0px);
      color: $caribbean-green;
      background-color: transparent;
      border: 1px solid $caribbean-green;
      border-radius: 0 0 10px 10px;
      overflow: hidden;
      transition: color $transition-short, background-color $transition-short, border $transition-short;

      &:hover,
      &:active {
        color: $white;
        background-color: $caribbean-green;
      }

      &.running {
        color: $supernova;
        border: 1px solid $supernova;

        &:hover,
        &:active {
          color: $white;
          background-color: $supernova;
        }
      }

      &.paused {
        color: $monza;
        border: 1px solid $monza;

        &:hover,
        &:active {
          color: $white;
          background-color: $monza;
        }
      }
    }
  }
</style>
