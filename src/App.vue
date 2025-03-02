<template>
  <div class="app-container">
    <RegistrationForm v-if="!isRegistered" @registration-complete="handleRegistrationComplete" />
    <MainInterface v-else />
  </div>
</template>

<script>
import axios from "axios";
import RegistrationForm from "./components/RegistrationForm.vue";
import MainInterface from "./components/MainInterface.vue";

const API_URL = "https://uniback-1.onrender.com"; // Укажи свой бэкенд

export default {
  components: {
    RegistrationForm,
    MainInterface,
  },
  data() {
    return {
      isRegistered: false,
      telegramId: null,
    };
  },
  async created() {
    await this.initializeTelegramUser();
    await this.checkUserRegistration();
  },
  methods: {
    async initializeTelegramUser() {
      if (window.Telegram?.WebApp) {
        const tg = window.Telegram.WebApp;
        const initData = tg.initDataUnsafe;
        this.telegramId = initData?.user?.id || null;
      }
    },
    async checkUserRegistration() {
      if (!this.telegramId) return;
      try {
        const response = await axios.get(`${API_URL}/api/main/${this.telegramId}`);
        this.isRegistered = response.data.isregistred;
      } catch (error) {
        console.error("Ошибка проверки регистрации:", error);
      }
    },
    handleRegistrationComplete() {
      this.isRegistered = true;
    },
  },
};
</script>
