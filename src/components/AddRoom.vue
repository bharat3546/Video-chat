<template>
  <div class="row roomForm">
    <div class="form-group mb-2">
      <input type="text" class="form-control" v-model="roomName" />
    </div>
    <button
      type="submit"
      :disabled="disable"
      class="btn btn-primary mb-2 createRoomBotton"
      @click="createNewRoom(roomName)"
    >
      Create Room
    </button>
  </div>
</template>

<script>
export default {
  name: "AddRoom",
  data() {
    return {
      roomName: ""
    };
  },
  props: ["disable"],
  methods: {
    createNewRoom(roomName) {
      if (!roomName) {
        alert("please provide a room name");
        return;
      }
      this.roomName = "";
      this.$emit("createNewRoom", roomName);
    }
  }
};
</script>

<style>
.roomForm {
  margin-left: auto;
  margin-right: auto;
  margin-top: 30px;
  width: 100%;
}
.createRoomBotton {
  color: #fff;
  background-color: #4d555f;
  border-color: #303840;
  padding: 8px;
  font-weight: bolder;
}
</style>
