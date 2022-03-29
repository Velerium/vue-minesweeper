<script>
export default {
  props: {
    show: Boolean,
    hints: Array,
    hintLimit: Number,
  }
}
</script>

<template>
  <Transition name="modalWin">
    <div v-if="show" class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <div class="modal-header">
            <slot name="header">Woo!!</slot>
          </div>

          <div class="modal-body">
            <slot name="body">You did it! Great job!
                <div v-if="hints.length < hintLimit && hintLimit - hints.length != 1">You had {{hintLimit - hints.length}} hints remaining too!</div>
                <div v-if="hintLimit - hints.length == 1">You had a hint remaining too!</div>
            </slot>
          </div>

          <div class="modal-footer">
            <slot name="footer">
              Click "New Game" to play again.
              <div>
                <button
                  class="modal-default-button"
                  @click="$emit('close'); $emit('disable')"
                >OK</button>
              </div>
            </slot>
          </div>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style>
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: table;
  transition: opacity 0.3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: top;
}

.modal-container {
  text-align: center;
  width: 400px;
  margin: 150px auto 0;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
}

.modal-header h3 {
  margin-top: 0;
  color: #42b983;
}

.modal-body {
  margin: 20px 0;
}

.modal-default-button {
  margin-top: 20px;
  width: 80px;

}

.modal-enter-from {
  opacity: 0;
}

.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-container,
.modal-leave-to .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
</style>