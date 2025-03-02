<template>
  <div class="app-container">
    <!-- Приветствие -->
    <transition name="fade">
      <div v-if="showGreeting" class="greeting-message">
        <h2>Привет!</h2>
      </div>
    </transition>

    <!-- Сообщение о регистрации -->
    <transition name="fade">
      <div v-if="showRegistrationText" class="registration-text">
        <h3>Чтобы продолжить, пройди регистрацию.</h3>
      </div>
    </transition>

    <!-- Форма регистрации -->
    <transition name="slide-up">
      <div v-if="showRegistrationForm" class="registration-container">
        <h2>Регистрация</h2>
        <form @submit.prevent="submitRegistration">
          <div class="form-group">
            <label for="lastName">Фамилия:</label>
            <input type="text" id="lastName" v-model="lastName" required />
          </div>
          <div class="form-group">
            <label for="firstName">Имя:</label>
            <input type="text" id="firstName" v-model="firstName" required />
          </div>
          <div class="form-group">
            <label for="middleName">Отчество:</label>
            <input type="text" id="middleName" v-model="middleName" />
          </div>
          <div class="form-group">
            <label for="birthDate">Дата рождения:</label>
            <input type="date" id="birthDate" v-model="birthDate" required />
          </div>
          <div class="form-group">
            <label for="birthTime">Время рождения:</label>
            <input type="time" id="birthTime" v-model="birthTime" required />
          </div>
          <button type="submit" class="submit-button">Зарегистрироваться</button>
        </form>
      </div>
    </transition>
  </div>
</template>

<script>
import axios from "axios";

const API_URL = "https://uniback-1.onrender.com"; // Бэкенд

export default {
  data() {
    return {
      showGreeting: true,
      showRegistrationText: false,
      showRegistrationForm: false,
      telegramId: null,
      lastName: "",
      firstName: "",
      middleName: "",
      birthDate: "",
      birthTime: "",
    };
  },
  methods: {
    async initializeTelegramUser() {
      try {
        if (window.Telegram?.WebApp) {
          const tg = window.Telegram.WebApp;
          const initData = tg.initDataUnsafe;
          if (!initData || !initData.user) {
            throw new Error("Ошибка получения Telegram ID");
          }
          this.telegramId = initData.user.id;
          tg.expand();
        } else {
          alert("Telegram Web App не поддерживается.");
        }
      } catch (error) {
        console.error("Ошибка инициализации Telegram:", error);
      }
    },

    async checkUserRegistration() {
      try {
        if (!this.telegramId) return;
        const response = await axios.get(`${API_URL}/api/main/${this.telegramId}`);
        if (response.data && response.data.isregistred) {
          localStorage.setItem("isRegistered", "true");
          this.$emit("registration-complete");
        } else {
          this.showRegistrationText = true;
          setTimeout(() => {
            this.showRegistrationText = false;
            this.showRegistrationForm = true;
          }, 2000);
        }
      } catch (error) {
        console.error("Ошибка при проверке регистрации:", error);
        this.showRegistrationText = true;
        setTimeout(() => {
          this.showRegistrationText = false;
          this.showRegistrationForm = true;
        }, 2000);
      }
    },

    async submitRegistration() {
      if (!this.telegramId) {
        alert("Ошибка: Telegram ID не получен.");
        return;
      }

      const userData = {
        tg_id: this.telegramId,
        surname: this.lastName,
        name: this.firstName,
        patronymic: this.middleName || "",
        birth_date: this.birthDate,
        birth_time: this.birthTime,
      };

      try {
        await axios.post(`${API_URL}/api/register`, userData);
        localStorage.setItem("isRegistered", "true");
        this.$emit("registration-complete");
      } catch (error) {
        console.error("Ошибка при регистрации:", error);
        alert("Не удалось зарегистрироваться. Попробуйте снова.");
      }
    },
  },
  async mounted() {
    await this.initializeTelegramUser();
    await this.checkUserRegistration();
  },
};
</script>

<style scoped>
.registration-container {
  width: 90%;
  max-width: 400px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}

@media (max-width: 768px) {
  .registration-container {
    width: 100%;
    padding: 15px;
  }
}
</style>
