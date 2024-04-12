<template>
  <div class="table-container">
    <table class="table-fixed w-full border-collapse border border-black">
      <tr
        v-for="(row, index) in rows"
        :key="index"
        class="border-collapse border border-black"
      >
        <td class="border-collapse border border-black text-center">
          <span
            v-for="(char, idx) in row.original.split('')"
            :key="idx"
            class="original-character"
          >
            {{ char }}
          </span>
        </td>
        <td class="border-collapse border border-black text-center">
          <span
            v-for="(char, idx) in row.modified"
            :key="idx"
            class="character inline-block p-1 m-0.5 cursor-pointer"
            :class="getClass(row, idx)"
            @click="selectChar(row, idx)"
          >
            {{ char }}
          </span>
        </td>
      </tr>
    </table>
    <div
      v-if="gameEnded"
      class="fixed inset-0 bg-gray-500 bg-opacity-75 flex justify-center items-center"
    >
      <div
        class="bg-white p-4 rounded-lg shadow-lg flex flex-col justify-center text-center items-center w-[400px] h-[400px]"
      >
        <img
          :class="{ rotating: rotateImage }"
          src="https://mikroten.github.io/roman-pismenka/yeah.png"
          class="w-[150px] h-[150px] rounded-full mb-3"
          alt=""
        />
        <h2 class="text-xl font-bold">Hurá!</h2>
        <p class="text-gray-700">Si veľmi šikovný!<br />Zaslužiš si oddych.</p>
        <p>Ale kapurkovú by si ešte mohol:</p>
        <div class="flex justify-center gap-3">
          <button
            @click="resetGame"
            class="mt-4 px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-700"
          >
            No daj
          </button>
          <a
            href="https://youtu.be/DmH_2yochqY?si=XofBmXsRwOwVfWAM&t=39"
            class="mt-4 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-700"
          >
            Skadze paru naberem
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import confetti from "canvas-confetti";

const rows = ref([]);
const gameEnded = ref(false);
const audio = new Audio("/roman-pismenka/yay.mp3");

const generateCharacterString = (
  length,
  characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()"
) => {
  return Array.from(
    { length },
    () => characters[Math.floor(Math.random() * characters.length)]
  ).join("");
};

const createRow = () => {
  const original = generateCharacterString(15);
  let modified = original.split("");
  const changeIndexes = new Set();

  // Change 4 characters
  while (changeIndexes.size < 4) {
    changeIndexes.add(Math.floor(Math.random() * modified.length));
  }

  changeIndexes.forEach((index) => {
    let newChar;
    do {
      newChar = generateCharacterString(1);
    } while (original.includes(newChar));
    modified[index] = newChar;
  });

  return { original, modified, selected: new Array(15).fill(null) };
};

const checkGameEnd = () => {
  const allCorrect = rows.value.every((row) =>
    row.modified.every(
      (char, idx) =>
        row.selected[idx] === "correct" || row.original.includes(char)
    )
  );
  if (allCorrect) {
    gameEnded.value = true;
    audio.play();
    confetti({
      particleCount: 300,
      spread: 300,
      origin: { y: 0.5 },
    });
  }
};

const resetGame = () => {
  rows.value = [];
  for (let i = 0; i < 28; i++) {
    rows.value.push(createRow());
  }
  gameEnded.value = false;
};

const selectChar = (row, idx) => {
  const char = row.modified[idx];
  const isCorrect = !row.original.includes(char);
  row.selected[idx] = isCorrect ? "correct" : "incorrect"; // Update selection status
  checkGameEnd();
};

const getClass = (row, idx) => {
  return row.selected[idx];
};

onMounted(() => {
  for (let i = 0; i < 28; i++) {
    rows.value.push(createRow());
  }
});
</script>

<style>
.original-character {
  margin-right: 8px; /* Adjust space between characters in the first column */
  display: inline-block;
}
.character {
  cursor: pointer;
  padding: 0.2rem;
  margin: 0.1rem;
  display: inline-block;
  color: black;
}
.correct {
  color: green;
}
.incorrect {
  color: red;
}
</style>
