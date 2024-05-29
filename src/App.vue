<template>
  <div>
    <p class="welcome-message">Silahkan Masukan Data Anda!</p>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title"/><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>
    <div class="articles-frame">
      <ul>
        <li v-for="article in articles" :key="article.id" class="article-item">
          <strong>{{ article.title }}</strong><br />
          {{ article.content }}<br />
          <button @click="edit(article)">Edit</button>
          <button @click="confirmDelete(article)">Hapus</button>
        </li>
      </ul>
    </div>
    <button @click="load">Load</button>
    <p class="clock">{{ currentDate }}</p>
    <p class="clock-time">{{ currentTime }}</p>
    <!-- Confirmation Modal -->
    <div v-if="showConfirmation" class="confirmation-modal">
      <p>Apakah ingin menghapus data ini?</p>
      <p><strong>{{ articleToDelete.title }}</strong></p>
      <p>{{ articleToDelete.content }}</p>
      <button @click="deleteArticle">Ya</button>
      <button @click="cancelDelete">Tidak</button>
    </div>
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
    const showConfirmation = ref(false);
    const articleToDelete = ref(null);

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
          // Update existing article in the list
          const index = articles.value.findIndex(article => article.id === form.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          // Add new article to the list
          articles.value.push(response.data);
        }
        // Reset form
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    function confirmDelete(article) {
      articleToDelete.value = article;
      showConfirmation.value = true;
    }

    async function deleteArticle() {
      if (articleToDelete.value) {
        try {
          await axios.delete(`http://localhost:3000/articles/${articleToDelete.value.id}`);
          articles.value = articles.value.filter(article => article.id !== articleToDelete.value.id);
          showConfirmation.value = false;
          articleToDelete.value = null;
        } catch (error) {
          console.error('Error deleting article:', error);
        }
      }
    }

    function cancelDelete() {
      showConfirmation.value = false;
      articleToDelete.value = null;
    }

    function deleteArticleConfirmation() {
      deleteArticle(); // Panggil fungsi deleteArticle yang sudah didefinisikan sebelumnya
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
      confirmDelete,
      deleteArticle,
      cancelDelete,
      showConfirmation,
      articleToDelete,
      currentTime,
      currentDate,
      deleteArticleConfirmation // Sertakan fungsi deleteArticleConfirmation di objek yang dikembalikan
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
 
.article-item {
  border-bottom: 1px solid gray;
  padding: 10px 0;
  background-color: rgb(0, 0, 0); /* Background color for each article */
  margin-bottom: 10px; /* Space between articles */
  padding: 15px; /* Padding inside each article */
  border-radius: 5px; /* Rounded corners for each article */
}

.article-item:last-child {
  border-bottom: none;
  margin-bottom: 0; /* Remove space after the last article */
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
  bottom: 50px; /* Position the date above the time */
}

.confirmation-modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgb(0, 0, 0);
  border: 1px solid black;
  padding: 20px;
  z-index: 1000;
}
</style>
