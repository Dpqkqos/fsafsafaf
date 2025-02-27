<template>
  <div class="app-container">
    <!-- Приветственное сообщение -->
    <transition name="fade" @after-leave="setShowRegistrationText">
      <div v-if="showGreeting" class="greeting-message">
        <h2>Привет!</h2>
      </div>
    </transition>

    <!-- Сообщение о регистрации -->
    <transition name="fade" @after-leave="setShowRegistrationForm">
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
            <input type="text" id="middleName" />
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

        <!-- Прогноз на день -->
        <div v-if="forecast" class="forecast-section">
          <h3>Ваш прогноз на день:</h3>
          <p>{{ forecast }}</p>
        </div>

        <!-- Таймер -->
        <div v-if="startTime" class="timer">
          <p>Время с момента регистрации: {{ formattedTime }}</p>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showGreeting: true,
      showRegistrationText: false,
      showRegistrationForm: false,
      lastName: "",
      firstName: "",
      middleName: "",
      birthDate: "",
      birthTime: "",
      forecast: "",
      startTime: null,
      timerInterval: null,
    };
  },
  computed: {
    formattedTime() {
      if (!this.startTime) return "00:00:00";
      const diff = Math.floor((Date.now() - this.startTime) / 1000);
      const hours = Math.floor(diff / 3600);
      const minutes = Math.floor((diff % 3600) / 60);
      const seconds = diff % 60;
      return `${String(hours).padStart(2, "0")}:${String(minutes).padStart(2, "0")}:${String(seconds).padStart(2, "0")}`;
    },
  },
  methods: {
    setShowRegistrationText() {
      setTimeout(() => {
        this.showRegistrationText = true;
      }, 3000);
    },
    setShowRegistrationForm() {
      setTimeout(() => {
        this.showRegistrationForm = true;
      }, 1000);
    },
    async submitRegistration() {
      const userData = {
        lastName: this.lastName,
        firstName: this.firstName,
        middleName: this.middleName,
        birthDate: this.birthDate,
        birthTime: this.birthTime,
      };

      try {
        const response = await fetch("https://uniback-production.up.railway.app/register", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(userData),
        });

        if (!response.ok) throw new Error("Ошибка регистрации");
        const data = await response.json();
        this.forecast = data.forecast;
        this.startTimer();
      } catch (error) {
        console.error("Ошибка при регистрации:", error);
        alert("Не удалось зарегистрироваться. Попробуйте снова.");
      }
    },
    startTimer() {
      this.startTime = Date.now();
      this.timerInterval = setInterval(() => {
        this.currentTime = Date.now();
      }, 1000);
    },
  },
  beforeUnmount() {
    if (this.timerInterval) {
      clearInterval(this.timerInterval);
    }
  },
  mounted() {
    setTimeout(() => {
      this.showGreeting = false;
    }, 3000);
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
html, body {
  height: 100vh;
  background: #fff;
  overflow: hidden; /* Убираем скроллбар */
}
.app-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  position: relative;
}
.greeting-message h2, .registration-text h3 {
  font-size: 2rem;
  background: linear-gradient(45deg, #f700ff, #e100ff, #6708ff);
  color: transparent;
  -webkit-background-clip: text;
}

/* Анимации */
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
.slide-up-enter-active {
  transition: transform 0.5s ease, opacity 0.5s ease;
}
.slide-up-enter-from {
  transform: translateY(50px);
  opacity: 0;
}
.slide-up-enter-to {
  transform: translateY(0);
  opacity: 1;
}

/* Стили формы */
.registration-container {
  width: 400px;
  padding: 20px;
  background: linear-gradient(45deg, #1f5bfe, #741efe, #6c11ff);
  border-radius: 10px;
}
.form-group {
  margin-bottom: 15px;
}
.form-group input {
  width: 100%;
  padding: 8px;
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}
.submit-button {
  width: 100%;
  padding: 10px;
  background: #fb0eff;
  color: #fff;
  cursor: pointer;
}
.submit-button:hover {
  background: #e62ee6;
}
</style>
