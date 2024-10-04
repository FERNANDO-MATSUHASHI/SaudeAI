<template>
  <div class="q-pa-md row justify-center chat-background">
    <q-card class="card">
      <q-card-section class="text-h6">Chat</q-card-section>

      <q-card-section virtual-scroll :virtual-scroll-item-size="50" style="max-height: 570px; overflow-y: auto;">
        <q-chat-message
          v-for="(message, index) in messages"
          :key="index"
          :name="message.name"
          :avatar="message.avatar"
          :stamp="message.stamp"
          :sent="message.sent"
          :text-color="message.textColor"
          :bg-color="message.bgColor"
        >
          <div v-html="message.content"></div>
        </q-chat-message>

        <!-- Exibe GIF de loading se estiver aguardando resposta -->
        <q-chat-message v-if="loading" name="Aguardando..." avatar="https://github.com/FERNANDO-MATSUHASHI/SaudeAI/blob/main/public/logo/Logo.png?raw=true" :sent="false" text-color="black" bg-color="amber">
          <div class="loading-message">
            <svg class="q-spinner" fill="currentColor" width="2rem" height="2rem" viewBox="0 0 120 30" xmlns="http://www.w3.org/2000/svg">
              <circle cx="15" cy="15" r="15">
                <animate attributeName="r" from="15" to="15" begin="0s" dur="0.8s" values="15;9;15" calcMode="linear" repeatCount="indefinite"></animate>
                <animate attributeName="fill-opacity" from="1" to="1" begin="0s" dur="0.8s" values="1;.5;1" calcMode="linear" repeatCount="indefinite"></animate>
              </circle>
              <circle cx="60" cy="15" r="9" fill-opacity=".3">
                <animate attributeName="r" from="9" to="9" begin="0s" dur="0.8s" values="9;15;9" calcMode="linear" repeatCount="indefinite"></animate>
                <animate attributeName="fill-opacity" from=".5" to=".5" begin="0s" dur="0.8s" values=".5;1;.5" calcMode="linear" repeatCount="indefinite"></animate>
              </circle>
              <circle cx="105" cy="15" r="15">
                <animate attributeName="r" from="15" to="15" begin="0s" dur="0.8s" values="15;9;15" calcMode="linear" repeatCount="indefinite"></animate>
                <animate attributeName="fill-opacity" from="1" to="1" begin="0s" dur="0.8s" values="1;.5;1" calcMode="linear" repeatCount="indefinite"></animate>
              </circle>
            </svg>
          </div>
        </q-chat-message>
      </q-card-section>
    </q-card>
    <!-- Card para Input e Botão -->
    <q-card class="card_button">
      <q-card-section>
        <div class="row items-center q-gutter-sm">
          <q-input
            v-model="userMessage"
            ref="userInput"
            filled
            placeholder="Digite sua mensagem..."
            @keyup.enter="sendMessage"
            class="col dark-input"
          />
          <q-btn
            round
            icon="send"
            color="primary"
            @click="sendMessage"
            :disable="!userMessage"
          />
        </div>
      </q-card-section>
    </q-card>

  </div>
</template>

<script>
import axios from "axios";
import './MainLayout.css';

export default {
  data() {
    return {
      userMessage: "",
      loading: false, // Novo estado para controle de loading
      messages: [
        {
          name: "Chat",
          avatar: "https://github.com/FERNANDO-MATSUHASHI/SaudeAI/blob/main/public/logo/Logo.png?raw=true",
          stamp: "Agora",
          content: "Olá! Como posso ajudar você?",
          sent: false,
          textColor: "black",
          bgColor: "amber",
        },
      ],
    };
  },
  mounted() {
    this.$refs.userInput.focus(); // Foca no input ao montar o componente
  },
  methods: {
    async sendMessage() {
      if (this.userMessage.trim()) {
        const messageToSend = this.userMessage;

        this.userMessage = "";

        this.messages.push({
          name: "Você",
          avatar: "https://github.com/FERNANDO-MATSUHASHI/SaudeAI/blob/main/public/logo/Person.png?raw=true",
          stamp: new Date().toLocaleTimeString(),
          content: messageToSend,
          sent: true,
          textColor: "white",
          bgColor: "primary",
        });

        this.loading = true; // Começa o loading

        try {
          const response = await axios.post(
            "https://chatllama-tw11.onrender.com/api/chat",
            {
              mensagem_usuario: messageToSend,
            }
          );

          this.messages.push({
            name: "Chat",
            avatar: "https://github.com/FERNANDO-MATSUHASHI/SaudeAI/blob/main/public/logo/Logo.png?raw=true",
            stamp: new Date().toLocaleTimeString(),
            content: response.data.resposta || "Resposta não disponível.",
            sent: false,
            textColor: "black",
            bgColor: "amber",
          });
        } catch (error) {
          console.error("Erro ao enviar mensagem:", error);
          this.messages.push({
            name: "Erro",
            avatar: "https://github.com/FERNANDO-MATSUHASHI/SaudeAI/blob/main/public/logo/Logo.png?raw=true",
            stamp: new Date().toLocaleTimeString(),
            content: "Não foi possível obter a resposta do servidor.",
            sent: false,
            textColor: "white",
            bgColor: "negative",
          });
        } finally {
          this.loading = false; // Para o loading
        }
      }
      this.$nextTick(() => {
        if (this.$refs.scrollArea) {
          this.$refs.scrollArea.scrollTop = this.$refs.scrollArea.scrollHeight;
        }
      });
    },
  },
};
</script>
