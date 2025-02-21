<template>
  <div id="app" class="w-full min-h-screen bg-gray-200 flex flex-col">
    <!-- Header -->
    <div class="w-full shadow-lg px-4 pt-2 pb-4 bg-indigo-600 z-30">
      <h1
        class="font-daruma text-[3.5vh] text-white bg-indigo-600 inline pb-2 px-6 rounded-md"
      >
        Chat Amigos
      </h1>
    </div>

    <!-- Main Chat Container -->
    <div class="flex flex-col flex-grow p-4 mt-4 overflow-hidden mb-32">
      <!-- Chat window -->
      <div
        class="flex-grow overflow-auto bg-white rounded-lg shadow-md p-4 max-w-4/5 w-full mx-auto"
      >
        <div
          v-if="queueStatus === 'waiting'"
          class="text-red-500 font-daruma tracking-wider text-[2vh]"
        >
          Waiting for another user to join...
        </div>
        <div
          v-if="queueStatus === 'connected'"
          class="mb-3 font-daruma text-[1.8vh] tracking-wide"
        >
          Someone has enter the chat, say "Hi!"
        </div>
        <div
          v-for="(message, index) in messages"
          :key="index"
          class="message mb-2"
        >
          <strong
            :class="[
              displayUsername(message.username) === 'You'
                ? 'text-blue-500'
                : 'text-red-500',
            ]"
          >
            {{ displayUsername(message.username) }}:
          </strong>
          <span>{{ message.content }}</span>
        </div>
      </div>

      <!-- Message Input -->
      <!-- Start New Chat Button -->
      <button
        v-if="queueStatus === 'notInQueue'"
        @click="requestNewChat"
        class="fixed bottom-6 left-1/2 transform -translate-x-1/2 bg-indigo-600 text-white px-6 py-2 rounded-md font-daruma text-[2.2vh] mb-3"
      >
        Start a New Chat
      </button>
      <div
        class="flex items-center justify-between mt-4 bg-white p-2 rounded-lg shadow-md fixed bottom-4 right-4 left-4 px-4 border-t border-gray-300 w-full max-w-4/5 mx-auto"
        v-else
      >
        <input
          v-model="newMessage"
          @keyup.enter="sendMessage"
          placeholder="Type a message..."
          class="w-full text-sm border border-gray-200 px-2 py-2 rounded-md"
        />
        <button
          @click="sendMessage"
          class="bg-indigo-600 text-white font-daruma md:text-[2.2vh] tracking-wider px-4 py-2 ml-2 rounded-md"
        >
          Send
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import io, { Socket } from "socket.io-client";

interface MessageType {
  username: string;
  content: string;
}

export default defineComponent({
  name: "App",
  setup() {
    const socket = io("http://localhost:3000"); // Replace with your server URL
    const newMessage = ref<string>("");
    const messages = ref<MessageType[]>([]);
    const username = `User${Math.floor(Math.random() * 1000)}`;
    const queueStatus = ref<"waiting" | "notInQueue" | "connected">(
      "notInQueue"
    );
    let roomName = ref<string>("");

    // Listen for new messages
    socket.on("newMessage", (message: MessageType) => {
      messages.value.push(message);
    });

    // Listen for when the user is connected to a new room
    socket.on("connectedToRoom", (room: string) => {
      roomName.value = room;
      queueStatus.value = "connected";
    });

    // Function to request a new chat
    const requestNewChat = () => {
      socket.emit("requestNewChat");
      queueStatus.value = "waiting";
    };

    // Function to send a message
    const sendMessage = () => {
      if (newMessage.value.trim() && roomName.value) {
        const message: MessageType = {
          username,
          content: newMessage.value.trim(),
        };
        socket.emit("sendMessage", message, roomName.value);
        newMessage.value = ""; // Clear input after sending
      }
    };

    // Function to display "You" or "Stranger"
    const displayUsername = (messageUsername: string): string => {
      return messageUsername === username ? "You" : "Stranger";
    };

    return {
      newMessage,
      messages,
      sendMessage,
      requestNewChat,
      displayUsername,
      queueStatus,
    };
  },
});
</script>

<style scoped>
/* Add your styles here */
</style>
