<template>
  <h1>Lists</h1>
  <div style="display: flex; gap: 20px;">
    <button class="btn btn-primary" @click="goToPrevPage" :disabled="currentPage === 1">Previous</button>
    <span class="fs-5 d-flex justify-content-center align-items-center">Page {{ currentPage }} of {{ totalPages }}</span>
    <button class="btn btn-primary" @click="goToNextPage" :disabled="currentPage === totalPages">Next</button>
  </div>
  
  <div class="d-flex">

    <div class="col-6">
      <div class="m-2">
        <label for="searchInput" class="form-label">Search:</label>
        <input type="text" class="form-control col-6" id="searchInput" v-model="searchTerm" />
      </div>
      <div class="main-content" style="height: 78vh; overflow-x: scroll;">
        <ul v-if="!isLoading && backData.length > 0" id="dataList">
          <li v-for="post in filteredData" :key="post.id">
            <h5>{{ post.id }}</h5>
            <h3 class="postTitle" @click="selectPost(post.id)" v-html="highlightMatches(post.title, searchTerm)" :style="{ color: isTitleBlue(post.title) ? 'blue' : 'inherit' }"></h3>
            <hr>
          </li>
        </ul>
      </div>
    </div>
 
    <div v-if="!isLoading && backData.length > 0" class="col-6 ps-5">
      <div class="m-2">
        <label for="searchInput" class="form-label">Search:</label>
        <input type="text" class="form-control col-6" id="searchInput" v-model="searchDescr" />
      </div>
      <div class="description-content" style="height: 78vh; overflow-x: scroll;">
        <div v-if="selectedPost">
          <h3 v-html="highlightMatches(selectedPost.title, searchDescr)"></h3>
          <p class="fs-5" v-html="highlightMatches(selectedPost.body, searchDescr)"></p>
        </div>
      </div>
    </div>

  </div>
  
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';


interface PostData {
  id: number,
  title: string,
  body: string,
  userId: number
}

let backData: PostData[] = []
let isLoading = ref(true)
let currentPage = ref(1)
let listsPerPage = ref(10);
let totalPages = ref(0);
let searchTerm = ref('');
let searchDescr = ref('');
let selectedPost = ref<PostData | null>(null);

const renderData = async () => {
  try {
    backData = [];
    for (let i = 0; i < 100; i++) {
      const response = await fetch(`http://localhost:3000/${i}`);
      const datas = await response.json();
      backData.push(datas);
      if(i == 0){
        selectedPost.value = datas
      }
    }
    totalPages.value = Math.ceil(backData.length / listsPerPage.value);
  } catch (error) {
    console.error('Error fetching data:', error);
  }
  isLoading.value = false;
};

const currentPageData = computed(() => {
  const startIndex = (currentPage.value - 1) * listsPerPage.value;
  const endIndex = currentPage.value * listsPerPage.value;
  return backData.slice(startIndex, endIndex);
});

const isTitleBlue = (title: string) => {
  return selectedPost.value && title.toLowerCase() === selectedPost.value.title.toLowerCase();
};

const filteredData = computed(() => {
  return currentPageData.value.filter(post =>
    post.title.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
    // post.body.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
    String(post.id).includes(searchTerm.value)
  );
});

const goToNextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value += 1;
  }
};

const goToPrevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value -= 1;
  }
};

const selectPost = (postId: number) => {
  selectedPost.value = backData.find(post => post.id === postId) || null;
};

const highlightMatches = (text: string, searchDescr: string) => {
  if (!searchDescr) return text;

  const regex = new RegExp(searchDescr, 'gi');
  return text.replace(regex, match => `<span style="background-color: yellow;">${match}</span>`);
};


onMounted(()=>{
  renderData().then()
}) 



</script>

<style scoped>
</style> 


