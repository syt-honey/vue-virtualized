<template>
  <div id="app">
    <v-virtualized-list :sourceList="sourceList">
      <template slot-scope="scope">
        <p>row name is: {{ scope.row.name }}</p>
        <p>row id is: {{ scope.row.id }}</p>
        <p>row index is: {{ scope.index }}</p>
        <p>row index is: {{ scope.index }}</p>
      </template>
    </v-virtualized-list>
  </div>
</template>

<script>
import VVirtualizedList from "./components/VVirtualizedList";

export default {
  name: "App",
  components: {
    VVirtualizedList
  },
  data() {
    return {
      sourceList: []
    };
  },
  mounted() {
    let lenOfName = 6;
    for (let i = 0; i < 100; ++i) {
      this.sourceList.push({
        name: this.genName(lenOfName),
        id: i
      });
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

