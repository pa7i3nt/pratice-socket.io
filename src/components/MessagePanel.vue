<template>
  <div>
    <div class="header">
      <status-icon :connected="user.connected" />{{ user.username }}
    </div>

    <ul class="messages">
      <li
        v-for="(message, index) in user.messages"
        :key="index"
        class="message">
        <div v-if="displaySender(message, index)" class="sender">
          {{ message.fromSelf ? "(yourself)" : user.username }}
        </div>
        {{ message.content }}
      </li>
    </ul>

    <form @submit.prevent="onSubmit" class="form">
      <textarea
        @keydown.enter.exact.prevent="onSubmit"
        v-model="input"
        placeholder="Your message..."
        class="input"
        @input="onTyping" />
      <button :disabled="!isValid" class="send-button">Send</button>
      <p id="typing-message"></p>
    </form>
  </div>
</template>

<script>
import socket from "../socket";
import StatusIcon from "./StatusIcon";

let typing = false;
let timeout = undefined;

export default {
  name: "MessagePanel",
  components: {
    StatusIcon,
  },
  props: {
    user: Object,
  },
  data() {
    return {
      input: "",
    };
  },
  methods: {
    onSubmit() {
      this.$emit("input", this.input);
      this.input = "";
    },
    displaySender(message, index) {
      return (
        index === 0 ||
        this.user.messages[index - 1].fromSelf !==
          this.user.messages[index].fromSelf
      );
    },
    onTyping(event) {
      if (socket.userID !== this.user.userID) {
        if (event.which != 13) {
          typing = true;
          socket.emit("typing", {
            to: this.user.userID,
            data: { typing: true, user: this.user },
          });
          clearTimeout(timeout);
          timeout = setTimeout(this.typingTimeOut, 1500);
        } else {
          clearTimeout(timeout);
          this.typingTimeOut();
        }
      }
    },
    // Updates the typing event
    typingTimeOut() {
      typing = false;
      socket.emit("typing", {
        to: this.user.userID,
        data: {
          typing: false,
          user: this.user,
        },
      });
    },
  },
  computed: {
    isValid() {
      return this.input.length > 0;
    },
  },
};
</script>

<style scoped>
.header {
  line-height: 40px;
  padding: 10px 20px;
  border-bottom: 1px solid #dddddd;
}

.messages {
  margin: 0;
  padding: 20px;
}

.message {
  list-style: none;
}

.sender {
  font-weight: bold;
  margin-top: 5px;
}

.form {
  padding: 10px;
}

.input {
  width: 80%;
  resize: none;
  padding: 10px;
  line-height: 1.5;
  border-radius: 5px;
  border: 1px solid #000;
}

.send-button {
  vertical-align: top;
}

#typing-message {
  text-align: right;
}
</style>
