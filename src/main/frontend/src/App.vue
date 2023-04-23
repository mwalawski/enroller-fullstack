<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername"></MeetingsPage>
    </div>

    <div v-else>
      <button @click="isLogging=true; message=''" type="submit" :class="isLogging ? '' : 'button button-outline'">{{ 'Logowanie' }}</button>
      <button @click="isLogging=false; message=''" type="submit" :class="isLogging ? 'button button-outline' : ''">{{ 'Rejestracja' }}</button>

      <div v-if="message" :class="isError ? 'alert' : 'info'">
        {{ message }}
      </div>

      <LoginForm v-if="isLogging" @login="(user) => logMeIn(user)"></LoginForm>
      <LoginForm v-else @login="(user) => register(user)" button-label="Załóż konto"></LoginForm>

    </div>
  </div>
</template>

<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from "axios";

export default {
  components: {LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      authenticatedUsername: '',
      isLogging: false,
      message:"",
      isError: false
    }
  },
  methods: {
    logMeIn(user) {
      axios.post('api/tokens', user)
          .then(response => {
            this.authenticatedUsername = user.login;
            this.message = "";
            const token = response.data.token;
            axios.defaults.headers.common['Authorization'] = 'Bearer ' + token;
          })
          .catch(response => {
            this.message = "Nie udało się zalogować użytkownika"
            this.isError = true;
          });
    },
    logMeOut() {
      this.authenticatedUsername = '';
      delete axios.defaults.headers.common.Authorization;
      this.message = "Użytkownik poprawnie wylogowany"
      this.isError = false;
    },
    register(user) {
      axios.post('api/participants', user)
          .then(response => {
            this.message = "Konto założone poprawnie"
            this.isError = false;
            this.isLogging = true;
          })
          .catch(response => {
            this.message = "Nie udało się żałożyć konta"
            this.isError = true;
          });
    }
  }
}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}
.alert {
  padding: 5px;
  margin: 5px;
  border: 1px solid #800000;
  background: #ee9093;
  font-size: 1em;
  max-width: 300px;
  text-align: center;
}

.info {
  padding: 5px;
  margin: 5px;
  border: 1px solid #00803c;
  background: #90ee9d;
  font-size: 1em;
  max-width: 300px;
  text-align: center;
}

</style>
