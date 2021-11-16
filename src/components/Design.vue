<template>
  <div class="container">
    <div class="my-image" :class="size" :style="getStyle(candidate)">
      This is some text
    </div>
    <!-- <svg width="400" height="400" viewbox="0 0 400 400">
      <g>
        <rect
          x="0"
          y="0"
          width="400"
          height="400"
          :style="getStyle(candidate)"
        />
        <text
          x="20"
          y="20"
          width="300"
          height="300"
          font-size="20"
          fill="black"
          text-anchor="middle"
        >
          This is some text
        </text>
      </g>
    </svg> -->
  </div>
</template>

<script>
export default {
  name: "Design",
  props: ["candidate", "size", "palette"],
  methods: {
    getStyle(candidate) {
      const style = {};
      for (let property in candidate.index) {
        const values = this.palette[property];
        style[property] = values[candidate.index[property]];
      }
      return style;
    },
  },
};
</script>

<style scoped>
.container {
  text-align: center;
  margin: 5px;
}
.my-image {
  margin-left: auto;
  margin-right: auto;
  border: 1px solid black;
  padding: 20px;
}
.small {
  width: 100px;
  height: 100px;
}
.large {
  width: 400px;
  height: 400px;
}
</style>
