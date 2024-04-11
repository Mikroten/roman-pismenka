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
            class="character"
            :class="getClass(row, idx)"
            @click="selectChar(row, idx)"
          >
            {{ char }}
          </span>
        </td>
      </tr>
    </table>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const rows = ref([]);

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

const selectChar = (row, idx) => {
  const char = row.modified[idx];
  const isCorrect = !row.original.includes(char);
  row.selected[idx] = isCorrect ? "correct" : "incorrect"; // Update selection status
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
