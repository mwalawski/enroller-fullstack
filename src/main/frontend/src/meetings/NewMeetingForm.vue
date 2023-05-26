<template>
  <form @submit.prevent="addNewMeeting(newMeeting)">
    <h3>Dodaj nowe spotkanie</h3>
    <label>Nazwa</label>
    <input type="text" v-model="newMeeting.title">
    <label>Opis</label>
    <textarea v-model="newMeeting.description"></textarea>
    <button>Dodaj</button>
    <span class="error" v-if="this.error">{{ this.message }}</span>
  </form>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      newMeeting: {participants: []},
      error: false,
      message: ""
    };
  },
  methods: {
    addNewMeeting(newMeeting) {
      if (this.newMeeting.title) {
        axios.post('api/meetings', newMeeting)
            .then(response => {
              this.error = false;
              this.message = "";
              this.$emit('added', this.newMeeting);
              this.newMeeting = {participants: []};
            })
            .catch(response => {
              this.error = true;
              this.message = "W tej chwili nie można dodać spotkania. Spróbuj ponownie później"
            })
      } else {
        this.error = true;
        this.message = "Spotkanie musi mieć nazwę!";
      }
    }
  }
}
</script>

<style scoped>
.error {
  color: #F00;
}
</style>
