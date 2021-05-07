<template>
  <div
    class="vue-virtualized"
    :style="{ paddingTop: startOffset + 'px', paddingBottom: endOffset + 'px' }"
  >
    <div
      v-for="(name, index) in shownList"
      :key="index"
      class="vue-virtualized--item"
      :style="{ height: rowHeight + 'px' }"
    >
      <div class="vue-virtualized--avatar">
        {{ getFirstCharOfName(name) }}
      </div>
      <div class="vue-virtualized--right">
        <span>{{ name }}</span>
        <span>This is row {{ startIndex + index }}</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "VueVirtaulized",
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
      default: () => 60
    }
  },
  data() {
    return {
      shownList: [],
      containerHeight: null,
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
  computed: {
    getFirstCharOfName() {
      return function (name) {
        return name.substr(0, 1);
      };
    }
  },

  mounted() {
    // 初始化 container 高度、可视区域展示的个数、startOffset、endOffset 等值
    this.init();

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
            // 更新 startIndex、endIndex、shownList、anchorItem、
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
        this.shownList.shift();
        this.shownList.push(this.sourceList[this.endIndex]);

        this.$nextTick(() => {
          this.cache.shift();
          const eleList = document.querySelectorAll(".vue-virtualized--item");
          this.cachePosition(
            Array.from(eleList)[eleList.length - 1],
            this.endIndex
          );
        });
      } else if (this.direction === "down") {
        // TODO 下拉有 bug
        this.shownList.pop();
        this.shownList.reverse();
        this.shownList.push(this.sourceList[this.startIndex - 1]);
        this.shownList.reverse();

        this.$nextTick(() => {
          this.cache.pop();
          const eleList = document.querySelectorAll(".vue-virtualized--item");
          this.cachePosition(
            Array.from(eleList)[0],
            this.startIndex - 1,
            false
          );
        });
      }

      this.startOffset = this.anchorItem.top;
      this.endOffset =
        (this.sourceList.length - this.endIndex) * this.rowHeight;
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
      this.endIndex = this.startIndex + this.visibleCount - 1;
    },
    /**
     * 缓存可视区域的 data
     */
    cachePosition(node, index, isTrail = true) {
      if (node) {
        const rect = node.getBoundingClientRect();
        const top = rect.top + window.pageYOffset;

        if (isTrail) {
          this.cache.push({
            index: index,
            top,
            bottom: top + this.rowHeight
          });
        } else {
          this.cache.reverse();
          this.cache.push({
            index: index,
            top,
            bottom: top + this.rowHeight
          });
          this.cache.reverse();
        }
      }
    },
    init() {
      this.$nextTick(() => {
        this.containerHeight = window.getComputedStyle(
          document.querySelector(".vue-virtualized")
        ).height;
        this.visibleCount = Math.ceil(
          parseInt(this.containerHeight) / this.rowHeight
        );
        this.startIndex = 0;
        this.endIndex = this.startIndex + this.visibleCount - 1;
        this.shownList = this.sourceList.slice(this.startIndex, this.endIndex);
        this.startOffset = 0;
        this.endOffset =
          (this.sourceList.length - this.endIndex) * this.rowHeight;

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
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.vue-virtualized--item {
  display: flex;
  align-items: center;
  padding-left: 15px;
  border: 1px solid #e0e0e0;
  border-bottom: none;
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
  font-size: 1.5em;
  background: rgb(244, 67, 54);
}

.vue-virtualized--right {
  display: flex;
  flex-direction: column;
}

span {
  text-align: start;
}
</style>