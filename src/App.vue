<template>
  <div id="app">
    <vue-virtualized :sourceList="sourceList" :overscan="overscan"></vue-virtualized>
  </div>
</template>

<script>
import VueVirtualized from "./components/VueVirtualized";

export default {
  name: "App",
  components: {
    VueVirtualized
  },
  data() {
    return {
      sourceList: [],
      overscan: 8
    };
  },
  mounted() {
    let lenOfName = 6;
    for (let i = 0; i < 100; ++i) {
      this.sourceList.push(this.genName(lenOfName));
    }
  },
  methods: {
    /**
     * caseType: 大写还是小写，0 大写，1 小写
     */
    genRandomChar(caseType) {
      const step = caseType === 0 ? 65 : 97;
      return String.fromCharCode(Math.random() * 26 + step);
    },
    genName(len) {
      let name = "";
      for (let i = 0; i < len; ++i) {
        if (name.length === 0) {
          name += this.genRandomChar(0);
        } else {
          name += this.genRandomChar(1);
        }
      }

      return name;
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100vw;
}

</style>
