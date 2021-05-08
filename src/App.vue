<template>
  <div id="app">
    <vue-virtualized :sourceList="sourceList"></vue-virtualized>
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
      sourceList: []
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
     * caseType: uppercase letter or lowercase letter，0 uppercase，1 lowercase
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

