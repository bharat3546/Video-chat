<template>
  <div class="col-md-6 box">
    <div class="roomTitle">
      <span v-if="loading"> Loading... {{ roomName }}</span>
      <span v-else-if="!loading && roomName"> Connected to {{ roomName }}</span>
      <span v-else>Select a room to get started</span>
    </div>
    <div class="row remote_video_container">
      <div id="remoteTrack" ref="remoteTrack"></div>
    </div>
    <div class="spacing"></div>
    <div class="row">
      <div id="localTrack"></div>
    </div>
    <button @click="reloadPage()" v-if="activeRoom">Leave Video call</button>
  </div>
</template>

<script>
import { EventBus } from "../Event";
import Twilio, { createLocalVideoTrack } from "twilio-video";
import axios from "axios";
export default {
  name: "Video",
  data() {
    return {
      loading: false,
      data: {},
      localTrack: false,
      remoteTrack: "",
      activeRoom: "",
      previewTracks: "",
      identity: "",
      roomName: null
    };
  },
  props: ["username"],
  created() {
    EventBus.$on("enterRoom", roomName => {
      this.createChat(roomName);
    });

    window.addEventListener("beforeunload", this.leaveRoomIfJoined);
  },
  watch: {
    activeRoom() {
      if (this.activeRoom) {
        this.$emit("activeroom");
      }
    }
  },
  methods: {
    reloadPage() {
      window.location.reload();
    },
    async getAccessToken() {
      return await axios.get(
        `http://localhost:3000/token?identity=${this.username}`
      );
    },
    // Trigger log events
    dispatchLog(message) {
      EventBus.$emit("new_log", message);
    },
    // Attach the Tracks to the DOM.
    attachTracks(tracks, container) {
      tracks.forEach(function(track) {
        container.appendChild(track.attach());
      });
    },
    // Attach the Participant's Tracks to the DOM.
    attachParticipantTracks(participant, container) {
      let tracks = Array.from(participant.tracks.values());
      this.attachTracks(tracks, container);
    },
    // Detach the Tracks from the DOM.
    detachTracks(tracks) {
      tracks.forEach(track => {
        track.detach().forEach(detachedElement => {
          detachedElement.remove();
        });
      });
    },
    // Detach the Participant's Tracks from the DOM.
    detachParticipantTracks(participant) {
      let tracks = Array.from(participant.tracks.values());
      this.detachTracks(tracks);
    },
    // Leave Room.
    leaveRoomIfJoined() {
      if (this.activeRoom) {
        this.activeRoom.disconnect();
      }
    },
    createChat(roomName) {
      this.loading = true;
      const VueThis = this;
      this.getAccessToken().then(data => {
        VueThis.roomName = null;
        const token = data.data.token;
        let connectOptions = {
          name: roomName,
          audio: true,
          video: { width: 400 }
        };
        // before a user enters a new room,
        // disconnect the user from they joined already
        this.leaveRoomIfJoined();

        // remove any remote track when joining a new room
        this.$refs.remoteTrack.innerHTML = "";
        Twilio.connect(token, connectOptions).then(room => {
          //   console.log(`Room Joined ${room}`);
          VueThis.dispatchLog(
            `Successfully joined a Room:${roomName} ${this.username}`
          );
          // set active Room
          VueThis.activeRoom = room;
          VueThis.roomName = roomName;
          VueThis.loading = false;
          // Attach the Tracks of the Room's Participants.
          room.participants.forEach(participant => {
            let previewContainer = this.$refs.remoteTrack;
            VueThis.attachParticipantTracks(participant, previewContainer);
          });
          // When a Participant joins the Room, log the event.
          room.on("participantConnected", participant => {
            VueThis.dispatchLog("Joining: '" + participant.identity + "'");
          });
          // When a Participant adds a Track, attach it to the DOM.
          room.on("trackAdded", (track, participant) => {
            VueThis.dispatchLog(
              participant.identity + " added track: " + track.kind
            );
            let previewContainer = this.$refs.remoteTrack;
            VueThis.attachTracks([track], previewContainer);
          });
          // When a Participant removes a Track, detach it from the DOM.
          room.on("trackRemoved", function(track, participant) {
            VueThis.dispatchLog(
              participant.identity + " removed track: " + track.kind
            );
            VueThis.detachTracks([track]);
          });
          // When a Participant leaves the Room, detach its Tracks.
          room.on("participantDisconnected", function(participant) {
            VueThis.dispatchLog(
              "Participant '" + participant.identity + "' left the room"
            );
            VueThis.detachParticipantTracks(participant);
          });
          // if local preview is not active, create it
          if (!VueThis.localTrack) {
            createLocalVideoTrack().then(track => {
              let localMediaContainer = document.getElementById("localTrack");
              localMediaContainer.appendChild(track.attach());
              VueThis.localTrack = true;
            });
          }
        });
      });
    }
  }
};
</script>

<style>
.remote_video_container {
  left: 0;
  margin: 0;
  border: 1px solid rgb(124, 129, 124);
}
#localTrack video {
  border: 3px solid rgb(124, 129, 124);
  margin: 0px;
  max-width: 50% !important;
  background-repeat: no-repeat;
}
.spacing {
  padding: 20px;
  width: 100%;
}
.roomTitle {
  border: 1px solid rgb(124, 129, 124);
  padding: 4px;
  color: dodgerblue;
}
</style>
