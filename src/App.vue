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
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden; /* Предотвращает прокрутку */
}

body {
  font-family: "Montserrat", sans-serif;
  line-height: 1.6;
  background: #fff; /* Белый фон для всего приложения */
}
.app-container {
  display: flex;
  justify-content: center; /* Центрирование по горизонтали */
  align-items: center; /* Центрирование по вертикали */
  position: relative;
  height: 100vh; /* Занимает всю высоту viewport */
  width: 100%; /* Занимает всю ширину viewport */
  padding: 20px; /* Отступы для контента */
  box-sizing: border-box; /* Включает padding и border в размер элемента */
  background: #fff; /* Белый фон */
  overflow: hidden; /* Предотвращает прокрутку при необходимости */
}
</style>
