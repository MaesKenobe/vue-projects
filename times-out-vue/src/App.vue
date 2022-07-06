<template>
  <header class="header"></header>
  <div class="content">
    <div class="container">
      <div class="row justify-content-center">
        <div class="col col-md-10">
          <h1 class="title">Time's out</h1>
          <div class="card__container">
            <timer_card v-for="timer in memoryTimers" :key="timer.timerId" :cardId="timer.timerId" :cardTitle="timer.timerTitle" :cardDescription="timer.timerDescription" :cardStatus="timer.timerStatus" :cardBuffer="timer.timerBuffer" :cardTimestamp="timer.timerTimestamp" @editTimer="editSpecificTimer" @removeTimer="removeSpecificTimer" @changeTimerStatus="changeSpecificTimer"/>
          </div>
          <div class="global__actions">
            <input type="button" class="button button--primary actions--add" value="+" @click="createTimer">
            <input type="button" class="button button--primary actions--remove-all" value="Remove all"  @click="clearStorage">
          </div>
        </div>
      </div>
    </div>
  </div>
  <footer class="footer">
    <span class="copyright">Cocomore<sup>©</sup> {{ currentTime.year }}</span>
    <span class="currentTime">{{ currentTime.day }}/{{ currentTime.month }}/{{ currentTime.year }}, {{ currentTime.hour }}:{{ currentTime.minutes }}:{{ currentTime.seconds }}</span>
  </footer>
  <dialog_card :open="dialogStatus == true ? 'open' : false" :targetId="targetId" :tempTimer="tempTimer" @closeDialog="closeThisDialog" @saveDialog="createSpecificTimer"></dialog_card>
</template>

<script>
  import timer_card from './components/timer_card.vue';
  import dialog_card from './components/dialog_card.vue';

  export default {
    name: 'App',
    components: {
      timer_card,
      dialog_card,
    },
    data () {
      return {
        dialogStatus: false,
        // Load by default the items in local storage.
        memoryTimers: this.loadStorage(),
        // Default timer object.
        targetId: "",
        tempTimer: {},
        // Default time values.
        currentTime: {
          now:              new Date(),
          currentTimestamp: new Date().getTime(),
          originTimestamp:  new Date().getTime(),
          buffer:           0,
          year:             String(new Date().getFullYear()),
          month:            String(new Date().getMonth() + 1).padStart(2, '0'),
          day:              String(new Date().getDate()).padStart(2, '0'),
          hour:             String(new Date().getHours()).padStart(2, '0'),
          minutes:          String(new Date().getMinutes()).padStart(2, '0'),
          seconds:          String(new Date().getSeconds()).padStart(2, '0'),
        }
      }
    },
    methods: {
      // STORAGE FUNCIONS.
      syncStorage() {
        // Save in local storage the array of objects.
        localStorage.setItem("memoryTimers", JSON.stringify(this.memoryTimers));
      },
      loadStorage() {
        // Function to load the elements in local storage, they are saved as JSON.
        let load = JSON.parse(localStorage.getItem("memoryTimers") || "[]");
        return load;
      },
      clearStorage() {
        // Clear from localstorage.
        let message ="Are you sure? There is no way to recover the timers.";

        if (confirm(message) == true) {
          // Clean local storage.
          localStorage.clear();
          // Set variables to default.
          this.memoryTimers = [];
        }
      },
      // SEARCH AND RECOVER TIMERS.
      searchSpecificTimer(timerId) {
        // Get position of this timer.
        return this.memoryTimers.map(function(e) { return e.timerId; }).indexOf(timerId);
      },
      retrieveSpecificTimer(index) {
        if (index > -1) {
          return this.memoryTimers[index];
        }
      },
      saveSpecificTimer(timer) {
        // Look if the timer already exists.
        let index = this.searchSpecificTimer(timer.timerId);

        // If position exists.
        if (index > -1) {
          this.memoryTimers = this.memoryTimers.map(e => e.timerId !== timer.timerId ? e : timer);
        }
        else {
        // Add the timer to the array.
          this.memoryTimers.push(timer);
        }

        // Save in local storage.
        this.syncStorage();
      },
      // CLOSE DIALOG.
      closeThisDialog() {
        // Close dialog.
        this.dialogStatus = false;
      },
      // CREATE TIMER.
      createTimer() {
        // Assign an Id.
        let targetId = 'timer-' + this.memoryTimers.length;
        this.targetId = targetId;
        this.tempTimer = {};
        // Launch the modal with the target Id.
        this.dialogStatus = true;
      },
      createSpecificTimer(timer) {
        // Call the save function.
        this.saveSpecificTimer(timer);
      },
      // EDIT TIMER.
      editSpecificTimer(cardId) {
        // Edit Specific timer.
        this.targetId = cardId;
        let index = this.searchSpecificTimer(cardId);
        this.tempTimer = this.retrieveSpecificTimer(index);
        this.dialogStatus = true;
      },
      // REMOVE TIMER.
      removeSpecificTimer(cardId) {
        // Remove specific timer.
        let message ="Are you sure? There is no way to recover the timer."

        if (confirm(message) == true) {
          // Look for the index of the timer based on its Id.
          let index = this.searchSpecificTimer(cardId);

          // If position exists.
          if (index > -1) {
            // Remove item from array.
            this.memoryTimers.splice(index, 1);

            // Save changes.
            this.syncStorage();
          }
        }
      },
      // CHANGE TIMER STATUS.
      changeSpecificTimer(cardId, status, newTimestamp) {
        let index = this.searchSpecificTimer(cardId);
        this.tempTimer = this.retrieveSpecificTimer(index);
        this.tempTimer.timerStatus = status;
        this.tempTimer.timerTimestamp = newTimestamp;
        this.saveSpecificTimer(this.tempTimer);
        this.tempTimer = [];
        console.log(this.memoryTimers);
        this.memoryTimers = this.loadStorage();
      },
      // UPDATE TIMERS.
      updateTimers() {

        // Footer clock.
        let now = new Date();
        this.currentTime = {
          current:  now,
          currentTimestamp: now.getTime(),
          originTimestamp:  this.currentTime.originTimestamp,
          buffer:           Math.round((this.currentTime.currentTimestamp - this.currentTime.originTimestamp) / 1000),
          year:             String(now.getFullYear()),
          month:            String(now.getMonth() + 1).padStart(2, '0'),
          day:              String(now.getDate()).padStart(2, '0'),
          hour:             String(now.getHours()).padStart(2, '0'),
          minutes:          String(now.getMinutes()).padStart(2, '0'),
          seconds:          String(now.getSeconds()).padStart(2, '0'),
        }

        // Update timers.
        if (this.memoryTimers.length > 0) {
          for (let i = 0; i < this.memoryTimers.length; i++) {
            let thisTimer = this.memoryTimers[i];
            let status = thisTimer.timerStatus;

            // Only update those who are running.
            if (status === 'running') {
              thisTimer.timerBuffer = thisTimer.timerBuffer + 1;
            }
          }
        }

        // Save changes.
        this.syncStorage();
      }
    },
    created() {
      let self = this;
      setInterval(function () {
        self.updateTimers();
      }, 1000);
    },
  }
</script>

<style lang="scss">
  @import '~@/assets/scss/base.scss';

  #app {
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    min-height: 100vh;
  }

  .content {
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    flex: 1 0 auto;
    width: 100%;
    padding: 50px 0;
  }

  .container {
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    flex: 1 0 auto;
    width: 100%;
  }

  .title {
    width: 100%;
    text-align: center;
    margin-bottom: 50px;
  }

  .card__container {
    display: grid;
    grid-template-columns: minmax(0, 1fr);
    grid-gap: 30px;
    margin: 0 auto 30px auto;

    @include breakpoint(md) {
      grid-template-columns: repeat(2, minmax(0, 1fr));
      margin: 0 auto 50px auto;
    }

    @include breakpoint(xl) {
      grid-template-columns: repeat(3, minmax(0, 1fr));
    }
  }

  .global__actions {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    gap: 15px;
  }

  .footer {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    flex-wrap: wrap;
    padding: 15px;

    .copyright {
      @extend h4;
    }

    .currentTime {
      width: 100%;
      text-align: center;
      @include font-size(12px);
      font-style: italic;
      font-weight: bold;

      @include breakpoint(xl) {
        text-align: right;
      }
    }
  }
</style>
