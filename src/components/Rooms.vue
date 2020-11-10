<template>
  <div class="col-md-3 rooms">
    <div
      class="room"
      v-for="room in rooms"
      v-bind:key="room.id"
      @click="enterRoom(room.name)"
    >
      {{ room.name }}
    </div>
    <AddRoom @createNewRoom="createNewRoom" :disable="disable" />
    <!-- Imported AddRoom component -->
  </div>
</template>

<script>
import { EventBus } from "../Event";
import AddRoom from "../components/AddRoom";
export default {
  name: "Rooms",
  props: ["disable"],
  data() {
    return {
      rooms: [
        { id: 1, name: "Conference Room1" },
        { id: 2, name: "Conference Room2" },
        { id: 3, name: "Conference Room3" }
      ],
      roomCount: 3,
      loading: false
    };
  },
  components: {
    AddRoom
  },
  methods: {
    enterRoom(room) {
      if (!this.disable) {
        EventBus.$emit("enterRoom", room);
      }
    },
    createNewRoom(data) {
      this.rooms.push({ id: this.roomCount++, name: data });
    }
  }
};
</script>

<style scoped>
.rooms > .room {
  border: 1px solid rgb(124, 129, 124);
  padding: 13px;
  margin: 3px 0px;
  color: ghostwhite;
}
.rooms {
  border: 1px solid rgb(64, 68, 64);
  cursor: pointer;
}
</style>
