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

const API_URL = "https://uniback-1.onrender.com"; // Бэкенд

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
        this.isRegistered = false; // Если ошибка, считаем пользователя незарегистрированным
      }
    },
    handleRegistrationComplete() {
      this.isRegistered = true;
    },
  },
};
</script>

<style>
.app-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  background: linear-gradient(45deg, #1f5bfe, #741efe, #6c11ff);
  background-size: 400% 400%;
  animation: gradient 4s ease infinite;
}

@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
</style>
