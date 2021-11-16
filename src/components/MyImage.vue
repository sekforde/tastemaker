<template>
  <div class="container">
    <div>
      Version {{ this.version }} Score {{ this.score }} Prediction
      {{ this.prediction }}
    </div>
    <div class="my-image" :style="getStyle()">This is some text</div>
    <div class="buttons">
      <button class="button no-button" @click="evaluateDesign(0)">No</button>
      <button class="button yes-button" @click="evaluateDesign(1)">Yes</button>
    </div>
    <div>{{ design }}</div>
    <div>{{ style }}</div>
  </div>
</template>

<script>
import * as brain from "brain.js";

const net = new brain.NeuralNetwork();
// const net = new brain.recurrent.LSTMTimeStep();

export default {
  name: "MyImage",
  data() {
    return {
      version: 1,
      palette: {
        backgrounds: ["red", "white", "black", "blue"],
        colors: ["red", "white", "black", "blue"],
        fontSizes: ["20px", "30px", "40px", "50px", "60px", "70px"],
        textAligns: ["left", "middle", "right"],
      },
      pattern: [],
      designValues: [],
      design: {
        backgroundColor: 0,
        color: 0,
        fontSize: 0,
        textAlign: 0,
      },
      style: {
        backgroundColor: "white",
        color: "black",
        fontSize: "20px",
        textAlign: "middle",
      },
      score: 0,
      prediction: 0,
    };
  },
  mounted() {
    this.createDesign();
  },
  methods: {
    getRandomValue: (arr) => Math.floor(Math.random() * arr.length),
    printDesignScore(d) {
      console.log(`${d.input} : ${d.output}`);
    },
    createDesign() {
      this.design = {
        backgroundColor: this.getRandomValue(this.palette.backgrounds),
        color: this.getRandomValue(this.palette.colors),
        fontSize: this.getRandomValue(this.palette.fontSizes),
        textAlign: this.getRandomValue(this.palette.textAligns),
      };
    },
    getStyle() {
      const style = {
        backgroundColor: this.palette.backgrounds[this.design.backgroundColor],
        color: this.palette.colors[this.design.color],
        fontSize: this.palette.fontSizes[this.design.fontSize],
        textAlign: this.palette.textAligns[this.design.textAlign],
      };
      return style;
    },
    evaluateDesign(score) {
      this.version++;
      this.score = score;
      this.trainModel(score);
      this.createDesign();
      this.predict();
    },
    trainModel(score) {
      const data = {
        input: this.design,
        output: {
          score,
        },
      };
      this.pattern.push(data);
      net.train(this.pattern, { iterations: 100, log: false });
    },
    predict() {
      this.prediction = net.run(this.design);
      // console.log("prediction", prediction);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container {
  text-align: center;
  width: 100%;
}
.my-image {
  margin-left: auto;
  margin-right: auto;
  border: 1px solid black;
  width: 400px;
  height: 400px;
  padding: 20px;
}
.buttons {
  margin-left: auto;
  margin-right: auto;
  border: 1px solid black;
  display: flex;
  width: 400px;
  flex-direction: row;
  margin-top: 50px;
}
.button {
  height: 100px;
  width: 100%;
  font-size: 40px;
  border-radius: 4px;
  margin: 5px;
  cursor: pointer;
}
.no-button {
  background-color: red;
}
.yes-button {
  background-color: green;
}
</style>
