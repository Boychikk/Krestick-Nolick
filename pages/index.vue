<script setup lang="ts">
const id = ref("");
const content = ref("");
const fielddata = ref(null);
const loading = ref(true);
const {data} = await useFetch(`https://express-sb.vercel.app/get-field-data`);
onMounted(async () => {
  console.log(data.value)
  fielddata.value = data.value?.data ?? [];
  loading.value = false;
});
const setValue = async () => {
  const newEl = {
    id: id.value,
    content: content.value
  }
  await $fetch("https://express-sb.vercel.app/set-field-value", {
    method: "POST",
    body: newEl
  })
  console.log(fielddata.value.findIndex(el => el.id == newEl.id))
  fielddata.value[fielddata.value.findIndex(el => el.id == newEl.id)] = newEl;
}

</script>

<template>
  <div class="center">
    <button @click="setValue">Рестарт</button>
    <div v-if="loading">Loading...</div>
    <div v-else class="grid">
      <div class="square" v-for="i in fielddata" :key="i.id" @click="id = i.id">
        {{ i.content }}
      </div>
    </div>
    <input v-model="id" />
    <input v-model="content" />

  </div>

</template>

<style>
body, html {
  height: 100%;
  width: 100%;
  overflow: hidden;
}

.center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  width: 600px;
  border: 2px solid black;
}

.square {
  border-right: 2px solid black;
  border-bottom: 2px solid black;
  text-align: center;
  height: 200px;
  word-break: break-all;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;;
}
.square:hover {
 background-color: gold;
  box-shadow: 0 0 20px 10px rgba(255, 215, 0, 0.7);

}

.square:nth-child(3n) {
  border-right: 0;
}

.square:nth-child(n+7) {
  border-bottom: 0;
}
button {
  width: 80px;
  height: 30px;
  border-radius: 4px;
}


</style>