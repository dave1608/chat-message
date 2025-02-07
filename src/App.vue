<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <!-- Campo para o nome-->
      <q-input
        outlined
        placeholder="Informe seu nome:"
        v-model="message.name"
        class="q-mt-sm q-pa-sm"
        dense
      ></q-input>

      
      <!-- Componente de chat -->
      <ChatComponent :messages="messages" :userActual="message.name" />

      <!-- Campo para enviar mensagem -->
      <q-input
        outlined
        @keyup.enter="send"
        placeholder="Digite uma mensagem"
        v-model="message.body"
        class="q-mt-xl q-pa-sm"
        dense
      >
        <template v-slot:append>
          <q-icon size="sm" name="send" @click="send" />
        </template>
      </q-input>
    </q-page-container>
  </q-layout>
</template>

<script>
import { ref, reactive, onMounted } from "vue";
import ChatComponent from "./components/ChatComponent.vue";
import Hub from "./Hub";

export default {
  name: "LayoutDefault",

  components: {
    ChatComponent,
  },

  setup() {
    const messages = ref([]); // Lista de mensagens
    const message = reactive({
      name: "", // Nome do usuário
      body: "", // Corpo da mensagem
    });
    const _hub = new Hub(); // Instância do Hub para conexão

    function send() {
      if (!message.body.trim()) return; // Evita enviar mensagens vazias
      _hub.connection.invoke("EnviarMensagem", message).catch((err) => {
        console.error("Erro ao enviar mensagem:", err);
      });
      message.body = ""; // Limpa o campo de mensagem após envio
    }

    onMounted(() => {
      _hub.connection
        .start()
        .then(() => {
          console.log("Connected");
          _hub.connection.on("ReceberMensagens", (msg) => {
            console.log(msg);
            messages.value.push(msg); // Adiciona a mensagem recebida na lista
          });
        })
        .catch((e) => console.error("Connection failed", e));
    });

    return {
      send,
      messages,
      message,
    };
  },
};
</script>
