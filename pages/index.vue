<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8>
      <v-card min-width="400">
        <v-snackbar v-model="snackbar" :timeout="6000" top>
          {{ message }}
          <v-btn color="pink" flat @click="snackbar = false">Закрити</v-btn>
        </v-snackbar>

        <v-card-title>
          <h1>Chat</h1>
        </v-card-title>
        <v-card-text>
          <v-form ref="form" v-model="valid" lazy-validation>
            <v-text-field v-model="name" :counter="16" :rules="nameRules" label="Ваше імя" required></v-text-field>

            <v-text-field v-model="room" :rules="roomRules" label="Введіть кімнату" required></v-text-field>

            <v-btn :disabled="!valid" color="primary" @click="submit">Вхід</v-btn>
          </v-form>
        </v-card-text>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapMutations } from "vuex";
export default {
  layout: "empty",
  head: {
    title: "Chat"
  },
  sockets: {
    connect: function() {
      console.log("socket connected");
    }
  },
  data: () => ({
    valid: true,
    snackbar: false,
    message: "",
    name: "",
    nameRules: [
      v => !!v || "Введіть імя",
      v => (v && v.length <= 16) || "Ім'я не повинно перевищувати 16 символів"
    ],
    room: "",
    roomRules: [v => !!v || "Введіть кімнату"]
  }),
  mounted() {
    const { message } = this.$route.query;
    if (message === "noUser") {
      this.message = "Введіть данні";
    } else if (message === "leftChat") {
      this.message = "Ви вийшли з чату";
    }

    this.snackbar = !!this.message;
  },
  methods: {
    ...mapMutations(["setUser"]),
    submit() {
      if (this.$refs.form.validate()) {
        const user = {
          name: this.name,
          room: this.room
        };

        this.$socket.emit("userJoined", user, data => {
          if (typeof data === "string") {
            console.error(data);
          } else {
            user.id = data.userId;
            this.setUser(user);
            this.$router.push("/chat");
          }
        });
      }
    }
  }
};
</script>
