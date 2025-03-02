<template>
  <div class="app-container">
    <!-- Приветственное сообщение -->
    <transition name="fade" @after-leave="showRegistrationText = true">
      <div v-if="showGreeting" class="greeting-message">
        <h2>Привет!</h2>
      </div>
    </transition>

    <!-- Сообщение о регистрации -->
    <transition name="fade" @after-leave="showRegistrationForm = true">
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
  watch: {
    showRegistrationText(newVal) {
      if (newVal) {
        setTimeout(() => {
          this.showRegistrationText = false; // Скрыть текст через 3 секунды
        }, 3000);
      }
    },
  },
  methods: {
    async initializeTelegramUser() {
      if (window.Telegram?.WebApp) {
        const tg = window.Telegram.WebApp;
        const initData = tg.initDataUnsafe;
        this.telegramId = initData.user.id;

        // Развернуть приложение на весь экран
        tg.expand();
      } else {
        alert("Telegram Web App не поддерживается.");
      }
    },
    async submitRegistration() {
      const userData = {
        tg_id: this.telegramId,
        surname: this.lastName,
        name: this.firstName,
        patronymic: this.middleName,
        birth_date: this.birthDate,
        birth_time: this.birthTime,
      };

      try {
        const response = await axios.post("https://uniback-1.onrender.com/api/register", userData);
        if (response.data.status === "success") {
          alert("Регистрация прошла успешно!");
        }
      } catch (error) {
        console.error("Ошибка при регистрации:", error);
        alert("Не удалось зарегистрироваться. Попробуйте снова.");
      }
    },
  },
  mounted() {
    setTimeout(() => {
      this.showGreeting = false; // Скрыть приветствие через 3 секунды
    }, 3000);

    this.initializeTelegramUser(); // Инициализация Telegram Web App
  },
};
</script>

<style scoped>
/* Основные стили */
* {
  font-family: "Montserrat", sans-serif;
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  background: #fff;
}

html,
body {
  width: 100%;
  height: 100vh;
  line-height: 1.6;
  background: #fff; /* Белый фон для всего приложения */
  overflow: hidden;
  -webkit-overflow-scrolling: touch; /* Для плавной прокрутки на iOS */
}

.app-container {
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  height: 100vh;
  width: 100%;
  padding: 10px; /* Уменьшим padding для мобильных устройств */
  overflow: hidden;
  background: #fff; /* Белый фон */
}

/* Градиентный текст */
.greeting-message h2,
.registration-text h3 {
  font-size: 1.2rem; /* Уменьшим размер шрифта для мобильных устройств */
  background: linear-gradient(45deg, #f70eff, #7700ff, #750cff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-align: center; /* Центрируем текст */
  margin: 0; /* Убираем лишние отступы */
}

/* Анимации */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.slide-up-enter-from {
  transform: translateY(20px);
  opacity: 0;
}

.slide-up-leave-to {
  transform: translateY(-20px);
  opacity: 0;
}

/* Стили формы регистрации */
.registration-container {
  width: 90%;
  max-width: 350px; /* Уменьшим максимальную ширину для мобильных устройств */
  padding: 15px; /* Уменьшим padding */
  background: linear-gradient(45deg, #1f5bfe, #741efe, #6c11ff);
  background-size: 400% 400%;
  animation: gradient 4s ease infinite;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  position: relative;
  top: 0;
  left: 0;
  margin: auto;
}

.form-group {
  margin-bottom: 10px; /* Уменьшим отступы */
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #fff;
  font-size: 0.8rem; /* Уменьшим размер шрифта */
}

.form-group input {
  width: 100%;
  padding: 8px; /* Уменьшим padding */
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  font-size: 0.8rem; /* Уменьшим размер шрифта */
}

.form-group input::placeholder {
  color: rgba(255, 255, 255, 0.7);
}

.submit-button {
  width: 100%;
  padding: 8px; /* Уменьшим padding */
  background: #fb0eff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s ease;
  font-size: 0.8rem; /* Уменьшим размер шрифта */
}

.submit-button:hover {
  background: #e62ee6;
}

/* Медиа-запросы для мобильных устройств */
@media (max-width: 768px) {
  .registration-container {
    width: 100%;
    padding: 10px; /* Уменьшим padding для маленьких экранов */
  }

  .form-group input {
    font-size: 0.7rem; /* Уменьшим размер шрифта */
  }

  .submit-button {
    font-size: 0.7rem; /* Уменьшим размер шрифта */
    padding: 6px; /* Уменьшим padding */
  }
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
