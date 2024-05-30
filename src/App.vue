<template>
  <div>
    <p class="welcome-message">Silahkan Masukan Data Anda!</p>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title"/><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>
    <div class="articles-frame">
      <div class="articles-grid">
        <div v-for="article in articles" :key="article.id" class="article-item">
          <strong>{{ article.title }}</strong><br />
          {{ article.content }}<br />
          <button @click="edit(article)">Edit</button>
          <button @click="deleteArticle(article)">Hapus</button>
        </div>
      </div>
    </div>
    <button @click="load">Load</button>
    <button @click="clearAll">Clear</button>
    <p class="clock">{{ currentDate }}</p>
    <p class="clock-time">{{ currentTime }}</p>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });

    const articles = ref([]);
    const currentTime = ref('');
    const currentDate = ref('');

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);
        if (form.id) {
          const index = articles.value.findIndex(article => article.id === form.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
        }
        resetForm();
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    async function deleteArticle(article) {
      try {
        const response = await axios.delete(`http://localhost:3000/articles/${article.id}`);
        articles.value = articles.value.filter(item => item.id !== article.id);
        resetForm();
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    async function clearAll() {
      try {
        for (const article of articles.value) {
          await axios.delete(`http://localhost:3000/articles/${article.id}`);
        }
        articles.value = [];
      } catch (error) {
        console.error('Error clearing articles:', error);
      }
    }

    function resetForm() {
      form.id = null;
      form.title = '';
      form.content = '';
    }

    function updateTime() {
      const now = new Date();
      const timeOptions = {
        timeZone: 'Asia/Jakarta',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
      };
      const dateOptions = {
        timeZone: 'Asia/Jakarta',
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      };
      currentTime.value = now.toLocaleTimeString('en-US', timeOptions);
      currentDate.value = now.toLocaleDateString('en-US', dateOptions);
    }

    onMounted(() => {
      load();
      updateTime();
      setInterval(updateTime, 1000);
    });

    return {
      form,
      articles,
      save,
      edit,
      load,
      deleteArticle,
      clearAll,
      currentTime,
      currentDate
    };
  }
};
</script>

<style scoped>
.welcome-message {
  border: 1px solid black;
  background-color: rgb(0, 0, 0);
  padding: 10px;
  display: inline-block;
  margin-top: 10px;
  color: white;
}

.articles-frame {
  display: flex;
  justify-content: center;
}

.articles-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  width: 100%;
  max-width: 1200px;
}

.article-item {
  border: 1px solid gray;
  padding: 10px;
  background-color: rgb(0, 0, 0);
  color: white;
  border-radius: 5px;
}

.clock, .clock-time {
  position: fixed;
  bottom: 10px;
  right: 10px;
  border: 1px solid rgb(0, 0, 0);
  background-color: rgb(0, 0, 0);
  padding: 10px;
  font-size: 18px;
  font-family: 'calibri', Courier, monospace;
  color: white;
}

.clock {
  bottom: 50px;
}
</style>
