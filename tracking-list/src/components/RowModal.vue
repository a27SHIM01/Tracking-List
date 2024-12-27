<script setup>
  import { Icon } from "@iconify/vue";

  const props = defineProps({
    show: Boolean,
    entry: Object
  })

  const emit = defineEmits(['remove', 'update'])
  
  function confirmRemove() {
    if (confirm("Are you sure you want to delete this entry?")) {
      emit('remove')
      emit('close')
    }
  }

  function saveAndExit() {
    emit('update')
    emit('close')
  }



</script>

<template>
  <Transition name="row-modal">
    <div v-if="show" class="row-modal-mask">
      <div class="row-modal-container">
        <div class="row-modal-header">
          <slot name="header"><h3>Edit Entry</h3></slot>
        </div>

        <div class="row-modal-body entry-info-align" >
          <slot name="body">
            <p>
              <label>Title:</label>
              <input v-model="entry.title" @keyup.enter="saveAndExit" required placeholder="No title">
            </p><br>
            <p>
            <label>Chapter:</label><input v-model="entry.chapter" @keyup.enter="saveAndExit" placeholder="No chapter">
            </p><br>
            <p>
            <label>Status:</label><input v-model="entry.read_status" @keyup.enter="saveAndExit" placeholder="No status">
            </p>
          </slot>
        </div>

        <div class="row-modal-footer">
          <slot name="footer">
            <button
              @click="confirmRemove">
              <Icon 
                icon="material-symbols:delete-outline-rounded" 
                width="36" 
                height="36" 
                style="color: #f00"/>
            </button>
            <button
              class="row-modal-default-button"
              @click="saveAndExit">
              <Icon 
                icon="material-symbols:save-outline-rounded" 
                width="36" 
                height="36" 
                style="color: #00008B" />
            </button>
          </slot>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style scoped>
  * {
    box-sizing: border-box;
  }
  .row-modal-mask {
    position: fixed;
    z-index: 9998;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    transition: opacity 0.3s ease;
  }

  .row-modal-container {
    width: 50%;
    margin: auto;
    padding: 20px 30px;
    background-color: #fff;
    border-radius: 2px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
    transition: all 0.3s ease;
  }

  .row-modal-header h3 {
    margin-top: 0;
    color: #42b983;
  }

  .row-modal-body {
    margin: 20px 0;
  }

  .entry-info-align {
    display: table;
    width: 100%;
  }

  .entry-info-align p {
    display: table-row;
  }

  .entry-info-align label {
    display: table-cell;
  }

  .entry-info-align input {
    display: table-cell;
    width: 100%;
  }

  .row-modal-default-button {
    float: right;
  }

  /*
  * The following styles are auto-applied to elements with
  * transition="row-modal" when their visibility is toggled
  * by Vue.js.
  *
  * You can easily play with the row-modal transition by editing
  * these styles.
  */

  .row-modal-enter-from {
    opacity: 0;
  }

  .row-modal-leave-to {
    opacity: 0;
  }

  .row-modal-enter-from .row-modal-container,
  .row-modal-leave-to .row-modal-container {
    -webkit-transform: scale(1.1);
    transform: scale(1.1);
  }
  
</style>
