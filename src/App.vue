<template>
  <div class="bg-gray-700 min-h-screen min-w-min text-center p-16 text-white">
    <div class="p-10">
      <h1 class="text-2xl">Try to guess the secret word.</h1>
      <button
        v-if="!gameStarted"
        class="text-4xl border-b-2 font-semibold pt-6"
        @click="startGame()"
      >
        Start
      </button>
    </div>

    <div v-if="gameStarted">
      <p class="p-6" v-if="!hasLost">Enter a letter:</p>
      <input
        v-if="!hasLost"
        class="bg-gray-700 border-gray-500 border-2 text-center"
        type="text"
        v-model="currentGuess"
        @keyup.enter="submitGuess()"
        autofocus
      />
      <div class="flex justify-center">
        <div
          :class="[
            item.show ? '' : 'opacity-0',
            'p-2 m-3 w-8 text-2xl select-none',
          ]"
          v-for="(item, index) in guessShowStatus"
          :key="index"
        >
          {{ item.letter }}
        </div>
      </div>
      <div class="flex justify-center">
        <div
          class="p-2 border-t-2 m-3 w-8 border-solid"
          v-for="(letter, index) in randomWord"
          :key="index"
        ></div>
      </div>
      <div class="p-6" v-if="!hasWon">
        <p class="p-4 text-gray-400">{{ triesLeft }} tries left!</p>
        <h1 v-if="wrongGuesses.length != 0">Wrong Guesses:</h1>
        <span class="text-3xl" v-for="(letter, i) in wrongGuesses" :key="i">
          {{ letter }},
        </span>
      </div>
    </div>

    <div v-if="hasWon">
      <p class="text-green-500">{{ message }}</p>
      <button class="border-b-2 pt-4" @click="restart">Back</button>
    </div>

    <div v-if="hasLost">
      <p class="text-red-400">{{ message }}</p>
      <button class="border-b-2 pt-4" @click="restart">Back</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      randomWord: [],
      gameStarted: false,
      currentGuess: "",
      wrongGuesses: [],
      correctGuesses: [],
      remainingGuesses: [],
      guessShowStatus: [],
      message: "",
      hasWon: false,
      hasLost: false,
      triesLeft: 8,
    };
  },

  methods: {
    async startGame() {
      const response = await fetch("https://random-words-api.vercel.app/word");
      const randomWord = await response.json();
      console.log(randomWord[0].word);
      this.randomWord = randomWord[0].word.toLowerCase().split("");
      this.remainingGuesses.push(...this.randomWord);
      this.guessShowStatus.push(...this.randomWord);
      this.guessShowStatus.forEach((letter, index) => {
        this.guessShowStatus[index] = {
          letter: letter,
          show: false,
        };
      });
      this.gameStarted = true;
    },

    submitGuess() {
      if (!/^[a-zA-Z]*$/g.test(this.currentGuess)) {
        alert("Letter only!");
        this.currentGuess = "";
      } else if (this.currentGuess.length < 1) {
        alert("Please enter a letter");
        this.currentGuess = "";
      } else if (this.currentGuess.length > 1) {
        alert("Only one letter at a time!");
        this.currentGuess = "";
      } else if (this.randomWord.includes(this.currentGuess.toLowerCase())) {
        this.correctGuesses.push(this.currentGuess.toLowerCase());
        this.remainingGuesses = this.remainingGuesses.filter(
          (l) => l != this.currentGuess.toLowerCase()
        );
        this.guessShowStatus.forEach((item) => {
          if (this.currentGuess.toLowerCase() == item.letter) {
            item.show = true;
          }
        });
        this.currentGuess = "";
      } else if (this.currentGuess.length == 1) {
        this.wrongGuesses.push(this.currentGuess.toLowerCase());
        this.currentGuess = "";
        this.triesLeft--;
      }

      if (this.remainingGuesses.length == 0) {
        this.message =
          "Congratulations, you've guessed the secret word correctly!";
        this.hasWon = true;
      }
      if (this.triesLeft == 0) {
        this.message = `You lost! The secret word is ${this.randomWord.join(
          ""
        )} `;
        this.hasLost = true;
      }
    },
    restart() {
      this.gameStarted = false;
      this.hasWon = false;
      this.hasLost = false;
      this.correctGuesses = [];
      this.remainingGuesses = [];
      this.wrongGuesses = [];
      this.randomWord = [];
      this.guessShowStatus = [];
      this.triesLeft = 8;
    },
  },
};
</script>
