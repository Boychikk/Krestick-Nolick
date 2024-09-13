<script setup lang="ts">
interface FieldData {
  id: number | string,
  content: string,
}

const fielddata = ref<FieldData[]>([]);
const loading = ref(true);
const winnerMsg = ref("");
onMounted(async () => {
  await getFieldData();
  loading.value = false;
});

const lines = [
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8],
  [0, 3, 6],
  [1, 4, 7],
  [2, 5, 8],
  [0, 4, 8],
  [2, 4, 6],
  [0, 4, 8],
  [2, 4, 6]
];

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
  winnerMsg.value = "";
  const newEl = {
    id,
    content: getSymbol.value,
  }
  await $fetch("https://express-sb.vercel.app/set-field-value", {
    method: "POST",
    body: newEl
  })
  fielddata.value[fielddata.value.findIndex(el => el.id == newEl.id)] = newEl;
  if (checkWinner()) {
    winnerMsg.value = `${checkWinner()} победил`;
    await resetField();
  }
}

const resetField = async () => {
  await $fetch("https://express-sb.vercel.app/reset-game", { method: "POST" })
  fielddata.value.map((el) => { return { ...el, content: -1 }})
}

const checkWinner = () => {
  let winner = "x";
  for (const el of lines) {
    el.forEach((item) => {
      if (fielddata.value[item]?.content !== winner) {
        winner = "0";
      }
    })
    if (winner === "x") break;
    el.forEach((item) => {
      if (fielddata.value[item]?.content !== winner) {
        winner = "";
      }
    })
    if (winner === "0") break;
  }
  return winner;
}
</script>

<template>
  <div class="center">
    <div>
      <div v-if="loading">Loading...</div>
      <div v-else class="grid">
        <div class="square" v-for="i in fielddata" :key="i.id" @click="setValue(i.id)">
          {{ i.content.replace("-1", "") }}
        </div>
      </div>
    </div>
    <button @click="resetField">Рестарт</button>
    <div class="winner">{{winnerMsg}}</div>
  </div>
</template>

<style>
body, html {
  height: 100%;
  width: 100%;
  overflow: hidden;
  margin: 0;
}

.center {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100vh;
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  width: 600px;
  border: 2px solid black;

  @media screen and (max-width: 700px){
    width: calc(100vw - 40px);
  }
}

.square {
  border-right: 2px solid black;
  border-bottom: 2px solid black;
  text-align: center;
  height: 200px;
  word-break: break-all;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 100px;
  font-weight: 700;

  @media screen and (max-width: 700px){
    height: 130px;
  }
}
.square:hover {
 background-color: rgba(0, 220, 130, 0.5);
  box-shadow: 0 0 20px 10px #108775;

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
.winner {
  font-size: 50px;
  font-weight: 700;
}

</style>