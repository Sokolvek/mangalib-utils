<script setup>
import { onMounted, ref } from "vue"
import data from "../json-fixer.json"

let origCollections = []
let collections = ref()
let search = ref("")
let all = []
let colls = []

const collectionNames = []
onMounted(async () => {

  getCollectionsNames(collectionNames)
  data.forEach((val) => all.push(...val.data))
  let removed = removeDuplicates()
  colls = await getCollections()
  origCollections = colls

  collections.value = colls

})

async function getCollections() {
  const resp = await fetch(`http://194.87.92.170:3000/mangalib`,
    {
      mode:"cors"
    })
  const res = await resp.json()
  return res
}

function removeDuplicates() {
  const res = []
  const removed = new Set([17156])
  all.forEach((val) => {
    if(removed.has(val.id)) 
      return 
    removed.add(val.id)
    res.push(val)
  })
  return res
}

function getCollectionsNames(arr) {
  const collectionNames = data.map((el) =>{
  el.data.map((el) => arr.push(el.name))

})

}

function downloadText(filename = 'data.txt') {
  const text = collections.value.map(item => JSON.stringify(item)).join('\n');
  const blob = new Blob([text], { type: 'text/plain' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

async function getNPageCollection(pageNum) {
  const resp = await fetch(`https://api.cdnlibs.org/api/collections?limit=24&page=${pageNum}&sort_by=newest`, {
    "headers": {
      "Site-Id": "1",
      "Content-Type": "application/json",
    },
    "referrer": "https://mangalib.me/",
    "method": "GET",
    "mode": "cors"
  })
  const res = await resp.json()
  return res
}

function goTo(url) {

  location.href = url
}

function handleSearch() {
  if(search.value === ""){
    collections.value = origCollections
    return
  }

  collections.value = collections.value.filter((col) => {
    return col.name.toLowerCase().includes(search.value.toLocaleLowerCase())
  })
}

</script>

<template>
  <div>
    <!-- <button @click="downloadText()">download</button> -->
    <input type="text" v-model="search" @input="handleSearch" class="search-bar">
    <div class="collection-list">
      <div v-for="collection in collections" class="collection-item">
        <h3>{{ collection.name }}</h3>
        <div class="cover-wrapper">
          <div v-for="imgLink in collection.previews">
            <img :src="imgLink.thumbnail" loading="lazy" alt="">
          </div>
        </div>
        <button @click="goTo(`https://mangalib.me/ru/collections/${collection.id}`)">
          Перейти к коллекции
        </button>
        <button @click="goTo(`https://mangalib.me/ru/user/${collection.user_id}`)">
          Ссылка на автора
        </button>
      </div>
    </div>
    <button v-if="search == ''" @click="goTo('https://youtu.be/dQw4w9WgXcQ?si=ZMPliRDojhIviqYv')">slash lib</button>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

.search-bar{
  margin-bottom: 50px;
}

.collection-list{
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;

}

.collection-item{
  border-radius: 10px;
  
  width: 380px;
  background: #1C1C1C;
  padding: 10px;
}

img {
  width: 125px;
  height: 194px;
}

.cover-wrapper{
  display: flex;
}

@media (max-width: 470px) {
  .collection-item{
    width: 300px;
  }
  img {
    width: 90px;
  }
}
</style>
