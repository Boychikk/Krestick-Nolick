<script setup lang="ts">
interface FieldData {
  id: number | string,
  content: string,
}

const fielddata = ref<FieldData[]>([]);
const loading = ref(true);
onMounted(async () => {
  await getFieldData();
  loading.value = false;
});

const getSymbol = computed(() => {
  let xAmount = 0;
  let oAmount = 0;
  for (const item of fielddata.value) {
    if (item.content.toLowerCase() === "x") xAmount += 1
    else if (item.content.toLowerCase() === "0") oAmount += 1;
  }
  return xAmount === oAmount ? "x" : "0";
})

const getFieldData = async () => {
  const result = await $fetch<{ data: FieldData[] }>(`https://express-sb.vercel.app/get-field-data`)
  fielddata.value = result.data
  setTimeout(getFieldData, 2500)
}

const setValue = async (id: number | string) => {
  const newEl = {
    id,
    content: getSymbol.value,
  }
  await $fetch("https://express-sb.vercel.app/set-field-value", {
    method: "POST",
    body: newEl
  })
  console.log(fielddata.value.findIndex(el => el.id == newEl.id))
  fielddata.value[fielddata.value.findIndex(el => el.id == newEl.id)] = newEl;
}

const resetField = async () => {
  await $fetch("https://express-sb.vercel.app/reset-game", { method: "POST" })
}
</script>

<template>
  <div class="center">
    <button @click="resetField">Рестарт</button>
    <div v-if="loading">Loading...</div>
    <div v-else class="grid">
      <div class="square" v-for="i in fielddata" :key="i.id" @click="setValue(i.id)">
        {{ i.content }}
      </div>
    </div>
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