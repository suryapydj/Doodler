<template>
  <div class="game-space">
    <h1>Welcome to Room {{ roomCode }}</h1>
    <p>Players in this room:</p>
    <ul>
      <li v-for="player in players" :key="player">{{ player }}</li>
    </ul>
    <button @click="start">Start</button>
    <button @click="getPrompt">Get Prompt</button>
    <button @click="changeDrawer">Change Drawer</button>
    <Canvas v-if="gameStarted"></Canvas>
  </div>
</template>
<script lang="ts">
import { defineComponent, onBeforeUnmount, ref } from "vue";
import { useRoute } from "vue-router";
import { useSocket } from "../socket.ts";

// this is important to understand https://www.matijanovosel.com/blog/define-component-vs-script-setup

export default defineComponent({
  setup() {
    const route = useRoute();
    //const router = useRouter();
    const roomCode = route.params.roomCode as string;
    const socket = useSocket();
    let gameStarted = ref(false)
    const players = ref<string[]>([]);
    let category: string;

    socket.emit("getRoomDetails", {roomCode});

    socket.on("roomDetails", (data) => {
      players.value = data.players;
      category = data.category;
    });

    const start = () => {
      socket.emit("startGame", {roomCode});
    };

    const getPrompt = () => {
      socket.emit("getPrompt", {roomCode, category});
    }

    const changeDrawer = () => {
      socket.emit("changeDrawer", {roomCode});
    }

    socket.on("newPrompt", (prompt) => {
      alert(prompt);
    });

    socket.on("gameStarted", () => {
      console.log("game started!");
      gameStarted.value = true;
    });

    onBeforeUnmount(() => {
      socket.off("roomDetails");
    });

    return {
      roomCode,
      players,
      start,
      gameStarted,
      getPrompt,
      changeDrawer,
    };
  },
});
</script>
<style> /* I'm trying to make it so that the canvas and player list are side-by-side but the styling isn't applying for some reason */
.game-space {
  display: flex,
}
</style>