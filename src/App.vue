<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <div class="messagesBox" ref="messagesBox">
    <div v-for="(message, i) in messages" :key="i">{{ message }}</div>
  </div>
  <input type="text" v-model="username" placeholder="Username" />
  <input
    type="text"
    v-model="entry"
    @keyup.enter="sendMessage"
    placeholder="Enter your message..."
  />
</template>

<script>
import { ref, watchEffect } from "vue";

export default {
  name: "App",
  setup() {
    const storedName = localStorage.getItem("username");
    const socket = new WebSocket("ws://web-socket-playground.herokuapp.com");
    const messages = ref([]);
    const username = ref(storedName || "Someone");
    const entry = ref("");
    const messagesBox = ref();

    watchEffect(() => {
      if (username.value.length) {
        localStorage.setItem("username", username.value);
      } else {
        username.value = "Someone";
      }
    });

    const sendMessage = () => {
      socket.send?.(`${username.value}: ${entry.value}`);
      entry.value = "";
    };

    socket.addEventListener("open", function (event) {
      socket.send(`${username.value} joined the server!`);
    });

    socket.addEventListener("message", function (event) {
      const data = JSON.parse(event.data);
      if (Array.isArray(data)) {
        messages.value = data;
      } else {
        messages.value.push(data);
      }
      messagesBox.scrollTop = messagesBox.scrollHeight;
    });

    return { entry, username, messages, sendMessage };
  },
};
</script>

<style scoped>
.messagesBox {
  height: 400px;
  overflow-y: scroll;
}
</style>