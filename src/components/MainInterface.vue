<template>
  <div class="app-container">
    <!-- Загрузчик -->
    <div v-if="loading" class="loader">Загрузка...</div>

    <!-- Основной контент -->
    <template v-else>
      <!-- Профиль -->
      <div class="profile-section">
        <h1 class="main-title">Личная карточка<span class="accent">✦</span></h1>
        <div class="profile-card">
          <img :src="user.avatar" class="user-avatar" alt="Аватар" />
          <div class="user-info">
            <h2 class="user-name">{{ user.fullName }}</h2>
            <div class="user-stats">
              <div class="stat-item">
                <span class="icon">✦</span>
                {{ user.daysOnPlatform }} {{ daysText }} на платформе
              </div>
              <div class="stat-item">
                <span class="icon">✦</span>
                Ваш запрос: {{ user.request }}
              </div>
            </div>
            <button @click="toggleRequestWindow" class="change-request-button">
              {{ showRequestModal ? 'Закрыть' : 'Изменить запрос' }}
            </button>
            <div v-if="showRequestModal" class="request-window">
              <div class="requests-list">
                <button
                  v-for="(request, index) in requests"
                  :key="index"
                  @click="selectRequest(request)"
                  class="request-item"
                >
                  {{ request }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Прогноз -->
      <div class="forecast-section">
        <h2 class="section-title">Прогноз на день</h2>
        <div class="forecast-card">
          <div class="forecast-content">
            <span class="forecast-icon">◎</span>
            <p>{{ forecast || 'Сегодня будет прекрасный день!' }}</p>
          </div>
        </div>
      </div>

      <!-- Эмоции -->
      <div class="emotions-section">
        <div class="emotions-header">
          <h2 class="section-title">Ведение эмоционального состояния<span class="accent">✦</span></h2>
          <button @click="toggleEmotionWindow" class="add-button">
            {{ showEmotionModal ? 'Закрыть' : '+ Добавить' }}
          </button>
          <div v-if="showEmotionModal" class="emotion-window">
            <textarea v-model="newEmotion" placeholder="Сегодня я чувствую..."></textarea>
            <button @click="addEmotion" class="save-btn">Сохранить</button>
          </div>
        </div>
        <div class="emotions-table">
          <div class="table-header">
            <div class="day-col">День</div>
            <div class="emotion-col">Эмоциональное состояние</div>
            <div class="action-col"></div>
          </div>
          <div v-for="(emotion, index) in reversedEmotions" :key="emotion.id" class="emotion-row">
            <div class="day-col">{{ totalEmotions - index }}</div>
            <div class="emotion-col">{{ emotion.state }}</div>
            <div class="action-col">
              <button @click="deleteEmotion(emotion.id)" class="delete-btn">🗑️</button>
            </div>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import axios from "axios";

const API_URL = "https://uniback-1.onrender.com"; // Бэкенд

export default {
  data() {
    return {
      loading: true,
      showEmotionModal: false,
      showRequestModal: false,
      newEmotion: "",
      forecast: "",
      user: {
        id: null,
        fullName: "Пользователь",
        avatar: "",
        emotions: [],
        daysOnPlatform: 0,
        request: "Любовь",
      },
      requests: ["Любовь", "Карьера", "Здоровье", "Финансы", "Саморазвитие", "Отношения"],
    };
  },
  computed: {
    reversedEmotions() {
      return [...this.user.emotions].reverse();
    },
    totalEmotions() {
      return this.user.emotions.length;
    },
    daysText() {
      const days = this.user.daysOnPlatform;
      if (days % 10 === 1 && days % 100 !== 11) return "день";
      if ([2, 3, 4].includes(days % 10) && ![12, 13, 14].includes(days % 100)) return "дня";
      return "дней";
    },
  },
  methods: {
    async initializeApp() {
      try {
        await this.initTelegramUser();
        await this.loadUserData();
        await this.loadUserRequest();
      } catch (error) {
        console.error("Ошибка инициализации:", error);
      } finally {
        this.loading = false;
      }
    },

    async initTelegramUser() {
      try {
        if (window.Telegram?.WebApp) {
          const tg = window.Telegram.WebApp;
          const initData = tg.initDataUnsafe;
          this.user.id = initData.user.id;
          this.user.avatar = initData.user.photo_url;
          tg.expand();
          tg.enableClosingConfirmation();
        } else {
          throw new Error("Telegram Web App не найден");
        }
      } catch (error) {
        console.error("Ошибка инициализации Telegram:", error);
      }
    },

    async loadUserData() {
      try {
        const response = await axios.get(`${API_URL}/user/${this.user.id}`);
        if (response.data) {
          this.user.fullName = `${response.data.first_name} ${response.data.middle_name || ""}`.trim() || "Пользователь";
          this.user.request = response.data.request;
        }

        const emotionsResponse = await axios.get(`${API_URL}/emotions/${this.user.id}`);
        this.user.emotions = emotionsResponse.data;
      } catch (error) {
        console.error("Ошибка загрузки данных:", error);
      }
    },

    async loadUserRequest() {
      try {
        const response = await axios.get(`${API_URL}/user/${this.user.id}`);
        if (response.data.request) {
          this.user.request = response.data.request;
        }
      } catch (error) {
        console.error("Ошибка при загрузке запроса:", error);
      }
    },

    async addEmotion() {
      if (!this.newEmotion.trim()) {
        this.showAlert("Поле эмоции не может быть пустым!");
        return;
      }

      try {
        const response = await axios.post(`${API_URL}/add_emotion`, {
          telegram_id: this.user.id,
          state: this.newEmotion,
        });

        this.user.emotions.push(response.data);
        this.newEmotion = "";
        this.showEmotionModal = false;
      } catch (error) {
        console.error("Ошибка при добавлении эмоции:", error);
        this.showAlert("Не удалось добавить эмоцию. Попробуйте снова.");
      }
    },

    async deleteEmotion(emotionId) {
      try {
        await axios.delete(`${API_URL}/emotion/${emotionId}`);
        this.user.emotions = this.user.emotions.filter((e) => e.id !== emotionId);
      } catch (error) {
        console.error("Ошибка удаления эмоции:", error);
        this.showAlert("Не удалось удалить эмоцию. Попробуйте снова.");
      }
    },

    async updateRequest(request) {
      try {
        await axios.post(`${API_URL}/update_request`, {
          telegram_id: this.user.id,
          request: request,
        });
        this.user.request = request;
      } catch (error) {
        console.error("Ошибка при обновлении запроса:", error);
      }
    },

    toggleEmotionWindow() {
      this.showEmotionModal = !this.showEmotionModal;
    },

    toggleRequestWindow() {
      this.showRequestModal = !this.showRequestModal;
    },

    selectRequest(request) {
      this.updateRequest(request);
      this.showRequestModal = false;
    },

    showAlert(message) {
      if (window.Telegram?.WebApp) {
        window.Telegram.WebApp.showAlert(message);
      } else {
        alert(message);
      }
    },
  },
  mounted() {
    this.initializeApp();
  },
};
</script>

<style>
@media (max-width: 768px) {
  .profile-card, .forecast-card, .emotions-table {
    width: 100%;
    margin: 10px 0;
  }

  .user-avatar {
    width: 80px;
    height: 80px;
  }

  .change-request-button, .add-button {
    font-size: 0.9rem;
    padding: 10px;
  }
}
</style>
