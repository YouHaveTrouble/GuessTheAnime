<template>
  <section class="anime">
    <div class="loader" v-if="gettingDataTask != null">
      <span>{{currentLoadingText}}</span>
    </div>
    <div class="animeWrap" v-if="gettingDataTask == null && animeData !== null">
      <img
          draggable="false"
          class="animeImage"
          :class="{blur: !answerRevealed}"
          :src="animeData.images.jpg.image_url"
          alt=""
      >
      <p>{{ displayData.synopsis }}</p>
      <div class="inputWrap">
        <input
            type="text"
            autocomplete="none"
            v-model="guessPhrase"
            :disabled="answerRevealed"
        >
        <div class="buttons">
          <input type="button" value="Guess" @click="guess()" :disabled="answerRevealed">
          <input type="button" value="Give up" @click="giveUp()" :disabled="answerRevealed">
        </div>

        <div v-if="answerRevealed" class="titleWrap">
          <p v-for="title in titles">{{ title }}</p>
          <input
              type="button"
              value="Guess another!"
              @click="() => {
              answerRevealed = false;
              getAnime();
            }"
          >
        </div>

      </div>

    </div>

  </section>
</template>

<script>
import levenshtein from 'js-levenshtein'
export default {
  name: "GuessingGame",
  data() {
    return {
      animeData: null,
      gettingDataTask: null,
      answerRevealed: false,
      titles: [],
      guessPhrase: "",
      displayData: {
        synopsis: ""
      },
      currentLoadingText: "",
      loadingText: [
          "Searching for a random anime",
          "Randomizing an anime",
          "Rolling the dice",
          "Animizing the random"
      ]
    }
  },
  methods: {
    async getAnime() {
      this.currentLoadingText = this.loadingText[Math.floor(Math.random()*this.loadingText.length)];
      this.animeData = null;
      this.guessPhrase = "";
      clearTimeout(this.gettingDataTask);

      const result = await fetch("https://api.jikan.moe/v4/random/anime");
      switch (result.status) {
        case 200:
          const json = await result.json();
          if (!this.isGuessable(json.data)) {
            this.setGetDataTask(1200);
            return;
          }
          this.animeData = json.data;
          this.titles = [];
          for (const id in this.animeData.titles) {
            const title = this.animeData.titles[id].title;
            if (!this.titles.includes(title)) this.titles.push(title);
          }
          this.setDisplayData(true);
          this.gettingDataTask = null;
          break;
        case 400:
          console.error("Error occured when getting data from the api")
          this.setGetDataTask(1200);
          break;
        case 429:
          alert("You're refreshing too much, wait a few seconds and try again!")
          break;
      }
    },
    setGetDataTask(delay) {
      clearTimeout(this.gettingDataTask);
      this.gettingDataTask = setTimeout(this.getAnime, delay);
    },
    isGuessable(animeDatajson) {
      // no synopsis
      if (animeDatajson.synopsis === null) return false;
      // synopsis less than 30 words
      if (animeDatajson.synopsis.split(" ").length < 50) return false;

      for (const genreId in animeDatajson.genres) {
        const genre = animeDatajson.genres[genreId].name;
        if (genre == null) continue;
        if (genre.toLowerCase().includes("hentai")) return false;
      }

      return true;
    },
    setDisplayData(censor) {
      this.displayData.synopsis = this.animeData.synopsis ?? "";
      this.displayData.synopsis = this.displayData.synopsis.replaceAll("&amp;", "&");
      if (!censor) return;
      for (const title in this.titles) {
        this.displayData.synopsis = this.displayData.synopsis.replaceAll(this.titles[title], "*****");
      }
    },
    guess() {
      for (const titleId in this.titles) {
        const title = this.titles[titleId];
        if (levenshtein(title.toLowerCase(), this.guessPhrase.toLowerCase()) <= 1) {
          this.answerRevealed = true;
          this.setDisplayData(false)
          return;
        }
      }
    },
    giveUp() {
      this.answerRevealed = true;
      this.setDisplayData(false)
    }
  },
  mounted() {
    this.getAnime();
  }
}
</script>

<style lang="scss" scoped>

section.anime {
  display: flex;
  width: 100%;
  max-width: 1280px;
  justify-content: center;
  align-items: center;
  transform: translate(-5px, -5px);
  .animeWrap {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 1.5rem;
    padding-inline: 1rem;
    .animeImage {
      padding: 0.5rem;
      max-width: min(100%, 300px);
      max-height: 300px;
      transition: filter 0.3s linear;
      &.blur {
        filter: blur(0.45rem);
      }
    }
    .titleWrap {
      text-align: center;
      input:last-child {
        margin-top: 0.5rem;
      }
    }
    .inputWrap {
      width: 100%;
      text-align: center;
      padding-bottom: 1rem;
      .buttons {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        padding-block: 0.5rem;
        gap: 0.5rem;
      }
      input[type=text] {
        background-color: #2c3e50;
        border: none;
        font-size: 1.2rem;
        padding-block: 0.5rem;
        color: #fff;
        text-align: center;
        &:disabled {
          filter: grayscale(1);
          cursor: default;
        }
      }
      input[type=button] {
        background-color: #2c3e50;
        font-size: 1.2rem;
        color: #ffffff;
        border: none;
        padding: 0.5rem;
        cursor: pointer;
        transition: filter 0.1s linear;
        &:not(:disabled):hover {
          filter: saturate(2);
        }
        &:disabled {
          filter: grayscale(1);
          cursor: default;
        }
      }
    }
  }
}

.loader {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  flex-direction: column-reverse;
  width: 100%;

  &:after {
    content: " ";
    display: block;
    width: 64px;
    height: 64px;
    margin: 8px;
    border-radius: 50%;
    border: 6px solid #fff;
    border-color: #fff transparent #fff transparent;
    animation: lds-dual-ring 1.2s linear infinite;
  }
}

@keyframes lds-dual-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

</style>