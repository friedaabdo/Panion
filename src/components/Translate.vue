<template>
  <div class="translate">
    <h1>Translate your text here</h1>
    <form @submit.prevent="translate">
      <div class="transBar">
        <input type="text" placeholder="words and phrases" v-model="input" />
        <button class="trans-button">Translate</button>
      </div>
    </form>
    <div class="history">
      <div v-for="(item, ind) in history" :key="item.input" class="word">
        <p v-if="this.history.length > 0">
          {{ item.input }} <span><i class="fas fa-globe"></i></span>
          {{ item.output }}
        </p>
        <button
          v-if="!item.saved"
          @click="addTranslation(ind)"
          class="save-button"
        >
          Save
        </button>
        <button v-else class="unsave-button">Saved!</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: { cityProps: Object, url: String },
  data() {
    return {
      input: "",
      output: "",

      history: [],
    };
  },
  methods: {
    async translate() {
      const api = process.env.VUE_APP_API;
      let url = `https://translation.googleapis.com/language/translate/v2?key=${api}&q= ${this.input}&source=${this.cityProps.source}&target=${this.cityProps.target}`;

      fetch(url, {
        method: "GET",
        headers: {
          "Content-Type": "application/json",
        },
      })
        .then((res) => res.json())
        .then((response) => {
          this.output = response.data.translations[0].translatedText;
          this.history.unshift({
            input: this.input,
            output: this.output,
            saved: false,
          });
          this.input = "";
        })
        .catch((error) => {
          console.log(
            "There was an error with the translation request: ",
            error
          );
        });
    },
    addTranslation(index) {
      this.history[index].saved = true;
      fetch(this.url + "/flashcards", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          original: this.history[index].input,
          translated: this.history[index].output,
        }),
      });
    },
  },
};
</script>

<style scoped>
.translate {
  margin-top: 75px;
  color: #12263a;
}

input {
  border-radius: 50px 0 0 50px;
  padding: 3px 10px;
  font-size: 20px;
  border: none;
  background-color: #f7a1a1;
  outline: none;
  width: 60%;
}
::placeholder {
  color: #f7fff7;
}
form {
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.trans-button {
  font-size: 20px;
  border-radius: 0 50px 50px 0;
  padding: 3px 10px;
  background-color: #f05d5e;
  border: none;
  color: #f7fff7;
  outline: none;
}
p {
  font-size: 20px;
}
.save-button {
  border: none;
  border-radius: 50px;
  font-size: 17px;
  padding: 3px 8px;
  background-color: #4ecdc4;
  margin-bottom: 20px;
  outline: none;
}
.unsave-button {
  border: none;
  border-radius: 50px;
  font-size: 17px;
  padding: 3px 8px;
  background-color: #ffe66d;
  margin-bottom: 20px;
  outline: none;
}
.word {
  border: 1px solid #12263a;
  margin: 10px;
  border-radius: 10px;
}
</style>
