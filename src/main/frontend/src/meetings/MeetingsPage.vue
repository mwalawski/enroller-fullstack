<template>
  <div>
    <NewMeetingForm @added="addNewMeeting($event)"></NewMeetingForm>
    <span v-if="meetings.length == 0">Brak zaplanowanych spotkań.</span>
    <h3 v-else>Zaplanowane zajęcia ({{ meetings.length }})</h3>
    <MeetingsList :meetings="meetings"
                  :username="username"
                  @attend="addMeetingParticipant($event)"
                  @unattend="removeMeetingParticipant($event)"
                  @delete="deleteMeeting($event)"></MeetingsList>
  </div>
</template>

<script>
import NewMeetingForm from "./NewMeetingForm";
import MeetingsList from "./MeetingsList";
import axios from "axios";

export default {
  components: {NewMeetingForm, MeetingsList},
  props: {username: String},
  data() {
    return {
      meetings: this.getMeetingsWithParticipants()
    };
  },
  methods: {
    getMeetingsWithParticipants() {
      let allMeetings = [];
      axios.get('api/meetings')
          .then(meetingsResponse => {
            this.error = false;
            this.message = "";
            meetingsResponse.data.forEach(meeting =>
                axios.get(`api/meetings/${meeting.id}/participants`)
                    .then(participantsResponse => {
                      meeting.participants = participantsResponse.data.map(participantData => participantData.login);
                      allMeetings.push(meeting)
                    })
            )
          })
      return allMeetings;
    },
    addNewMeeting(meeting) {
      // handled in child component NewMeetingForm
      this.meetings = this.getMeetingsWithParticipants() //need to reload list to retrieve id
    },
    addMeetingParticipant(meeting) {
      axios.post(
          `api/meetings/${meeting.id}/participants`,
          {"login":this.username})
          .then(response => {
            meeting.participants.push(this.username);
          })
    },
    removeMeetingParticipant(meeting) {
      axios.delete(`api/meetings/${meeting.id}/participants/${this.username}`)
          .then(response => {
            meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
          })
    },
    deleteMeeting(meeting) {
      axios.delete(`api/meetings/${meeting.id}`)
          .then(response => {
            this.meetings.splice(this.meetings.indexOf(meeting), 1);
          })
    },
  }
}
</script>
