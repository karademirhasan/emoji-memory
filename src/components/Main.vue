<template>
  <div class="Main">
    <div class="game-header">
      <div class="info-list">
        <div class="list-start">
          <div class="item item-game-difficulty" v-if="activeEmoji.emoji">
            game difficulty: {{ activeEmojiSize.value.label }}
            {{ activeEmojiSize.value.desc }} /
            {{ activeEmojiSize.value.value }} emojis
          </div>
          <div class="item">
            {{ correctCounter }} / {{ emojis ? emojis.length : "" }}
          </div>
          <div class="item" v-if="activeEmoji.emoji">
            you are looking for {{ activeEmoji.emoji }}
          </div>
        </div>
        <div class="list-end">
          <div class="item">
            <select
              class="emoji-size-select"
              v-model="selectedEmojiSize.value"
            >
              <option v-for="item of emojiOptions" :value="item" :key="item">
                {{ item.label }}
              </option>
            </select>
          </div>

          <div class="item">clicked {{ clickCounter }} times</div>
          <button class="restart-button" @click="restartGame()">
            restart 😤
          </button>
          <div class="item"></div>
        </div>
      </div>
    </div>

    <div class="list">
      <div
        class="item"
        @click="clickEmoji(item, $event)"
        v-for="item in emojis"
        :key="item.no"
      >
        <div
          class="emoji-card"
          :class="item.status ? item.status : 'no-status'"
        >
          <div class="front">{{ item.emoji }}</div>
          <div class="back">?</div>
        </div>
      </div>
    </div>
  </div>
  <div
    class="modal-overlay"
    :class="{ active: isGameCompleted && isActiveModal }"
  >
    <div class="modal-content">
      <div class="modal-header">
        <div @click="closeModal()" class="close-modal">👊</div>
      </div>
      <div class="game-result">
        <span class="smiley"> 🥳 </span>
        <span class="text">
          yeah! <br />
          you have completed! <br />
          clicked {{ clickCounter }} times
        </span>
      </div>
      <div class="game-actions">
        <button class="restart-button" @click="restartGame()">
          new game 🥴
        </button>
      </div>
    </div>
  </div>
</template>

  <script setup>
import { ref, reactive, watch } from "vue";

import listEmojis from "@/src/data/full-emoji-list.json";
const emojiOptions = [
  { value: 11, label: "🐣", desc: "hatching chick" },
  { value: 22, label: "🐥", desc: "baby chick" },
  { value: 33, label: "🐔", desc: "chicken" },
  { value: 44, label: "🦅", desc: "eagle" },
  { value: 55, label: "🦩", desc: "flamingo" },
  { value: 66, label: "🐋", desc: "whale" },
  { value: 77, label: "🦖", desc: "T-Rex" },
  { value: 88, label: "🦄", desc: "unicorn" },
  { value: 99, label: "🐉", desc: "dragon" },
];
const emojis = ref([]);
const activeEmoji = ref({});
const clickCounter = ref(0);
const correctCounter = ref(0);
const selectedEmojiSize = reactive({ value: emojiOptions[0] });
const activeEmojiSize = reactive({ value: emojiOptions[0] });
const isActiveModal = ref(true);
const isGameCompleted = ref(false);
const getRandomItem = (array) => {
  const newArray = array.filter((item) => item.status !== "correct");

  return newArray[Math.floor(Math.random() * newArray.length)];
};

const closeModal = () => {
  isActiveModal.value = false;
};

const shuffleArray = (array) => {
  let currentIndex = array.length,
    randomIndex;

  // While there remain elements to shuffle...
  while (currentIndex != 0) {
    // Pick a remaining element...
    randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex--;

    // And swap it with the current element.
    [array[currentIndex], array[randomIndex]] = [
      array[randomIndex],
      array[currentIndex],
    ];
  }

  return JSON.parse(
    JSON.stringify(array.slice(0, selectedEmojiSize.value.value))
  );
};

const resetEmojis = () => {
  emojis.value = shuffleArray([
    ...listEmojis["Activities"],
    ...listEmojis["Animals & Nature"],
    ...listEmojis["Objects"],
    ...listEmojis["Smileys & People"],
    ...listEmojis["Symbols"],
    ...listEmojis["Travel & Places"],
  ]);
};

resetEmojis();

const changeEmoji = async () => {
  activeEmoji.value = await getRandomItem(emojis?.value);
};

changeEmoji();

const clickEmoji = (item, event) => {
  const clickedEmojiIndex = emojis.value.findIndex(
    (element) => element.no === item.no
  );
  if (emojis.value[clickedEmojiIndex].status == "correct") return;
  clickCounter.value = clickCounter.value + 1;
  if (item.no === activeEmoji.value.no) {
    emojis.value[clickedEmojiIndex].status = "correct";
    correctCounter.value = correctCounter.value + 1;
    if (checkGameCompleted()) {
      isGameCompleted.value = true;

      return;
    }
    changeEmoji();
  } else {
    emojis.value[clickedEmojiIndex].status = "wrong";
    setTimeout(() => {
      emojis.value[clickedEmojiIndex].status = "";
    }, 500);
  }
};

const checkGameCompleted = () => {
  return correctCounter.value == emojis.value.length;
};

const restartGame = () => {
  resetEmojis();
  changeEmoji();
  clickCounter.value = 0;
  correctCounter.value = 0;
  isGameCompleted.value = false;
  isActiveModal.value = true;
  activeEmojiSize.value = selectedEmojiSize.value
};

watch(activeEmoji, () => {
  if (activeEmoji.value) {
    let cursorSvg = `url("data:image/svg+xml;utf8, <svg xmlns='http://www.w3.org/2000/svg' width='64' height='96' fill='white' style='font-size: 30px'><text y='30'>${activeEmoji.value.emoji}</text></svg>"), auto`;
    let faviconSvg = `data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>${activeEmoji.value.emoji}</text></svg>`;
    document.body.style.cursor = cursorSvg;
    document.querySelector("link[rel~='icon']").href = faviconSvg;
  } else {
    document.body.style.cursor = "auto";
  }
});
</script>

<style lang="scss" >
@import url("https://fonts.googleapis.com/css2?family=Fredoka:wght@300;400;500;600;700&display=swap");
* {
  user-select: none;
}

$color-red: #d51c1c;
body {
  background: #000;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100vw;
  margin: 0;
  font-family: "Fredoka", sans-serif;
  overflow-x: hidden;
}
.Main {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  width: 100vw;
  overflow-x: hidden;
  margin-top: 40px;
}
.list {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  width: 90%;
  .item {
    display: flex;
    justify-content: center;
    align-items: center;
    min-width: calc((100% / 11) - 3rem);
    height: 3.5rem;
    margin: 1.5rem;
    @media only screen and (max-width: 800px) {
      min-width: calc((100% / 6) - 3rem);
    }
  }
}

.emoji-card {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 2.4rem;
  border-radius: 6px;
  background: rgba(#121212, 0.9);
  box-shadow: 0 0 3px #d3d3d330;
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transform-origin: center right;
  transition: all 300ms ease-in-out;
  &:hover {
    background: rgba(#121212, 0.3);
  }
  &.correct {
    transform: translateX(-100%) rotateY(-180deg);
    transition: all 300ms ease-in-out;
    background: rgb(0 0 0);
    box-shadow: none;
    .front {
      opacity: 0.5;
    }
  }
  &.wrong {
    transform: translateX(-100%) rotateY(-180deg);
    transition: all 300ms ease-in-out;
  }
  .front,
  .back {
    backface-visibility: hidden;
    position: absolute;
    font-weight: 100;
  }
  .front {
    transform: rotateY(180deg);
    transition: all 400ms ease-in-out;
  }
}
.restart-button {
  all: unset;
  display: flex;
  align-items: center;
  justify-content: center;
  background: $color-red;
  padding: 0 1rem;
  border-radius: 6px;
  color: white;
  font-size: 1.6rem;
  transition: all 400ms ease-in-out;
  height: 64px;
  line-height: 2rem;
  &:hover {
    opacity: 0.2;
    transition: all 400ms ease-in-out;
  }
}
.emoji-size-select {
  all: unset;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 64px;
  border-radius: 6px;
  font-size: 2.6rem;
  background: $color-red;
  color: white;
  text-align: center;
  option {
    all: unset;
    font-size: 2.6rem;
  }
}
.game-header {
  display: flex;
  width: 90%;
  .info-list {
    display: flex;
    width: 100%;
    justify-content: space-between;
    flex-direction: row;
    margin-bottom: 20px;
    padding: 0 1.5rem;
    @media only screen and (max-width: 800px) {
      flex-direction: column;
    }
    .list-start {
      display: flex;
      flex-direction: column;
      text-align: left;
    }
    .list-end {
      display: flex;
      flex-direction: column;
      text-align: right;
    }
    .item {
      width: 100%;
      font-size: 2rem;
      margin-bottom: 1.5rem;

      &.item-game-difficulty {
        font-size: 1rem;
      }
    }
  }
}
.modal-overlay {
  display: none;
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
  background: rgb(0 0 0 / 66%);
  align-items: center;
  justify-content: center;
  &.active {
    display: flex;
  }
  .modal-content {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    width: 280px;
    height: 240px;
    background: $color-red;
    border-radius: 6px;
    padding: 30px 10px;
    position: relative;
    .modal-header {
      display: flex;
      justify-content: flex-end;
      position: absolute;
      right: 4px;
      top: 0;

      .close-modal {
        font-size: 1.6rem;
        &:hover {
          opacity: 0.6;
        }
      }
    }

    .game-result {
      .smiley {
        font-size: 3rem;
        padding-bottom: 10px;
      }
      font-size: 1.4rem;
      display: flex;
      flex-direction: column;
    }
    .game-actions {
      display: flex;
      justify-content: center;
      .restart-button {
        background: black;
      }
    }
  }
}
</style>
