<template>
  <div
    class="vue-virtualized"
    :style="{ paddingTop: startOffset + 'px', paddingBottom: endOffset + 'px' }"
  >
    <div
      v-for="(row, index) in shownList"
      :key="index"
      class="vue-virtualized--item"
      :style="{ height: rowHeight ? rowHeight + 'px' : 'auto' }"
    >
      <slot :row="row" :index="index + startIndex"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "VVirtaulizedList",
  props: {
    sourceList: {
      type: Array,
      default: () => []
    },
    overscan: {
      type: Number,
      default: () => 8
    },
    rowHeight: {
      type: Number,
      default: () => 100
    }
  },
  data() {
    return {
      shownList: [],
      screenHeight: null,
      startIndex: 0,
      endIndex: 0,
      startOffset: 0,
      endOffset: 0,
      anchorItem: {
        index: 0,
        top: 0,
        bottom: 0
      },
      cache: [],
      visibleCount: 0,
      direction: ""
    };
  },
  watch: {},

  created() {
    // Return to the top when refreshing the page
    setTimeout(() => window.scrollTo(0, 0), 150);
  },

  mounted() {
    // Initial height of container, the count of viewable area、startOffset、endOffset etc
    this.init();

    // Set scroll listener to update data
    let before = 0;
    window.addEventListener(
      "scroll",
      () => {
        const after = window.document.body.scrollTop
          ? window.document.body.scrollTop
          : window.document.documentElement.scrollTop;

        if (after > before) {
          if (after > this.anchorItem.bottom && this.anchorItem.bottom) {
            this.direction = "up";
            // update startIndex, endIndex, shownList, anchorItem, etc
            this.updateBoundaryIndex(after);
            this.updateData();
          }
        } else {
          if (after < this.anchorItem.top && this.anchorItem.top) {
            this.direction = "down";
            this.updateBoundaryIndex(after);
            this.updateData();
          }
        }
        before = after;
      },
      false
    );
  },
  methods: {
    updateData() {
      if (this.direction === "up") {
        this.shownList.reverse();
        this.shownList.pop();
        this.shownList.reverse();
        this.shownList.push(this.sourceList[this.endIndex - 1]);
        this.currentShowList = this.shownList;
      } else if (this.direction === "down") {
        if (this.startIndex > -1) {
          this.shownList.pop();
          this.shownList.reverse();
          this.shownList.push(this.sourceList[this.startIndex]);
          this.shownList.reverse();
          this.currentShowList = this.shownList;
        } else {
          // In theory, this is unreachable
          return;
        }
      }

      this.$nextTick(() => {
        this.startOffset = this.anchorItem.top;
        this.endOffset =
          (this.sourceList.length - this.endIndex) * this.rowHeight +
          (this.visibleCount * this.rowHeight - parseInt(this.screenHeight));

        if (this.endIndex > this.cache.length) {
          this.$nextTick(() => {
            const eleList = document.querySelectorAll(".vue-virtualized--item");
            this.cachePosition(
              Array.from(eleList)[eleList.length - 1],
              this.endIndex - 1
            );
          });
        }
      });
    },
    updateBoundaryIndex(scrollTop = 0) {
      const anchorItem = this.cache.find((item) => {
        return item.bottom >= scrollTop;
      });

      if (!anchorItem) {
        return;
      }

      this.anchorItem = { ...anchorItem };
      this.startIndex = this.anchorItem.index;
      this.endIndex = this.startIndex + this.visibleCount;
    },
    /**
     * Cache data in the visible area
     */
    cachePosition(node, index) {
      if (node) {
        const rect = node.getBoundingClientRect();
        const top = rect.top + window.pageYOffset;

        this.cache.push({
          index: index,
          top,
          bottom: top + this.rowHeight
        });
      }
    },
    init() {
      this.$nextTick(() => {
        this.screenHeight = window.getComputedStyle(
          document.querySelector(".vue-virtualized")
        ).height;
        this.visibleCount = Math.ceil(
          parseInt(this.screenHeight) / this.rowHeight
        );

        this.startIndex = 0;
        this.endIndex = this.startIndex + this.visibleCount;
        this.shownList = this.sourceList.slice(this.startIndex, this.endIndex);
        this.currentShowList = this.shownList;
        this.startOffset = 0;
        this.endOffset =
          (this.sourceList.length - this.endIndex) * this.rowHeight +
          (this.visibleCount * this.rowHeight - parseInt(this.screenHeight));

        this.$nextTick(() => {
          const eleList = document.querySelectorAll(".vue-virtualized--item");
          Array.from(eleList).map((ele, index) => {
            this.cachePosition(ele, index);
          });
          this.anchorItem = this.cache[0];
        });
      });
    }
  }
};
</script>

<style scoped>
.vue-virtualized {
  /* TODO: The height after using flex is not the specified height */
  /* display: flex;
  flex-direction: column; */
  height: 100vh;
}

.vue-virtualized--item {
  padding-left: 15px;
  border: 1px solid #e0e0e0;
  border-bottom: none;
  box-sizing: border-box;
}

.vue-virtualized--item:last-child {
  border-bottom: 1px solid #e0e0e0;
}

.vue-virtualized--avatar {
  width: 40px;
  height: 40px;
  margin-right: 25px;
  text-align: center;
  line-height: 40px;
  border-radius: 50%;
  color: #fff;
  font-size: 17px;
  background: rgb(244, 67, 54);
}

p {
  margin: 0;
  text-align: start;
}
</style>