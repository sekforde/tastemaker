<template>
  <div id="app">
    <div class="panel center">
      <h2>Suggested Design</h2>
      <Design :candidate="candidate" size="large" :palette="palette" />
      <div class="buttons">
        <button
          class="button no-button"
          :style="getButtonStyle(0)"
          @click="evaluateDesign(0)"
        >
          1
        </button>
        <button
          class="button no-button"
          :style="getButtonStyle(1)"
          @click="evaluateDesign(0.25)"
        >
          2
        </button>
        <button
          class="button no-button"
          :style="getButtonStyle(2)"
          @click="evaluateDesign(0.5)"
        >
          3
        </button>
        <button
          class="button no-button"
          :style="getButtonStyle(3)"
          @click="evaluateDesign(0.75)"
        >
          4
        </button>
        <button
          class="button no-button"
          :style="getButtonStyle(4)"
          @click="evaluateDesign(1)"
        >
          5
        </button>
      </div>
    </div>
    <div class="panel right">
      <h2>20 Candidates sorted</h2>
      <div v-for="suggestion in suggestions" :key="suggestion.index">
        <Design
          :candidate="suggestion.candidate"
          size="small"
          :palette="palette"
        />
        <div>{{ suggestion.prediction.score }}</div>
        <div>{{ suggestion.difference }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import Design from "./components/Design.vue";
import * as brain from "brain.js";

const net = new brain.NeuralNetwork();
// const net = new brain.recurrent.LSTMTimeStep();

export default {
  name: "App",
  components: {
    Design,
  },
  data() {
    return {
      version: 1,
      score: 0,
      prediction: 0,
      settings: {},
      colorPalette: [
        [100, 100, 100],
        [100, 100, 100],
        [100, 100, 100],
        [100, 100, 100],
        [100, 100, 100],
      ],
      candidate: {
        index: {
          backgroundColor: 0,
          // fill: 0,
          color: 2,
          fontSize: 3,
          textAlign: 1,
        },
        normal: {
          backgroundColor: 0,
          // fill: 0,
          color: 0,
          fontSize: 0,
          textAlign: 0,
        },
      },
      palette: {
        backgroundColor: ["red", "white", "black", "blue"],
        // fill: ["red", "white", "black", "blue"],
        color: ["red", "white", "black", "blue"],
        fontSize: [20, 30, 40, 50, 60, 70],
        textAlign: ["left", "center", "right"],
      },
      suggestions: [],
      pattern: [],
    };
  },
  mounted() {
    // const go = async () => {
    //   const data = await fetch("http://colormind.io/api/", {
    //     method: "POST",
    //     body: JSON.stringify({
    //       model: "default",
    //     }),
    //   })
    //     .then((result) => result.json())
    //     .catch(console.log);
    //   this.colorPalette = data.result;
    // };
    // go();
  },
  methods: {
    getRand: (limit) => Math.floor(Math.random() * limit),
    getRandomValue: (arr) => Math.floor(Math.random() * arr.length),
    generateCandidate() {
      const c = {
        index: {},
        normal: {},
      };
      for (let property in this.palette) {
        const values = this.palette[property];
        c.index[property] = this.getRand(values.length);
        c.normal[property] = c.index[property] / (values.length - 1);
      }
      return c;
    },
    evaluateDesign(score) {
      this.version++;
      this.score = score;
      this.trainModel(score);
      this.makeSuggestions();
      // create new candidate
      this.candidate = this.suggestions[0].candidate;
      this.prediction = this.predict(this.candidate);
    },
    trainModel(score) {
      const data = {
        input: this.candidate.normal,
        output: {
          score,
        },
      };
      this.pattern.push(data);
      net.train(this.pattern, { iterations: 1000, log: false });
      // net.train([data], { iterations: 1000, log: false });
    },
    predict(candidate) {
      return net.run(candidate.normal);
    },
    compare(design1, design2) {
      const propertyCount = Object.keys(design1).length;
      let differenceCount = 0;
      for (let property in design1) {
        // console.log(design1[property], design2[property]);
        if (design1[property] !== design2[property]) {
          differenceCount++;
        }
      }
      // console.log(differenceCount, propertyCount);
      return differenceCount / propertyCount;
    },
    makeSuggestions() {
      this.suggestions = [];
      for (let i = 0; i < 50; i++) {
        const candidate = this.generateCandidate();
        const difference = this.compare(this.candidate.index, candidate.index);
        this.suggestions.push({
          index: i,
          candidate,
          prediction: this.predict(candidate),
          difference,
        });
      }
      // this.suggestions.sort((a, b) => b.difference - a.difference);
      // this.suggestions.sort((a, b) => a.difference - b.difference);
      this.suggestions.sort((a, b) => b.prediction.score - a.prediction.score);
    },
    getButtonStyle(i) {
      const c = this.colorPalette[i];
      return {
        backgroundColor: `rgb(${c[0]}, ${c[1]}, ${c[2]})`,
      };
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
}
.panel {
  border: 1px solid black;
  flex-grow: 1;
}
.buttons {
  margin-left: auto;
  margin-right: auto;
  display: flex;
  width: 400px;
  flex-direction: row;
  margin-top: 50px;
  margin-bottom: 50px;
}
.button {
  height: 100px;
  width: 100px;
  font-size: 40px;
  border-radius: 4px;
  margin: 5px;
  cursor: pointer;
}
.no-button {
  background-color: grey;
}
.yes-button {
  background-color: green;
}
</style>
