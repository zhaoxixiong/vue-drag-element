<template>
  <div class="e-move" ref="eMove" :style="positions">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "VueDragElement",
  props: {
    options: {
      type: Object,
      default: () => {
        return {};
      }
    }
  },
  data() {
    return {
      doc: document,
      wrap: null,
      evs: {},
      startPoint: {},
      defaultPoint: {}, // transform point record
      tCurrentPoint: { x: 0, y: 0 },
      startTouchInfo: {},
      dis: { x: 0, y: 0 },
      numReg: /^\d+$/,
      platform: this.getPlatform(),
      wrapSize: { x: 0, y: 0 },
      screenceSize: { x: 0, y: 0 }, // target size x => width, y => height
      positionRange: { x: [0, 0], y: [0, 0] },
      position: {},
      isTransformSupport: this.cssSupported("transform"),
      translateRangeMap: {
        left: {
          name: "left",
          index: 0,
          type: "x"
        },
        top: {
          name: "top",
          index: 0,
          type: "y"
        },
        right: {
          name: "right",
          index: 1,
          type: "x"
        },
        bottom: {
          name: "bottom",
          index: 1,
          type: "y"
        }
      },
      edgeSpace: this.options.edgeSpace ? [...this.options.edgeSpace] : []
    };
  },
  created() {
    // init data in created
    const dataName = ["startPoint", "defaultPoint", "position"];
    for (let item of dataName) {
      this[item] = this.options.position ? { ...this.options.position } : {};
    }
    // if transform support reset startPoint atrribute value
    if (this.isTransformSupport) {
      for (let key in this.startPoint) {
        this.startPoint[key] = 0;
      }
    }
    this.evs = {
      START: this.platform ? "touchstart" : "mousedown",
      MOVE: this.platform ? "touchmove" : "mousemove",
      END: this.platform ? "touchend" : "mouseup"
    };
    this.screenceSize = {
      x: document.documentElement.clientWidth || document.body.clientWidth,
      y: document.documentElement.clientHeight || document.body.clientHeight
    };

    const [x0 = 0, x1 = 0, y0 = 0, y1 = 0] = this.options.edgeSpace
      ? this.edgeSpace.length === 1
        ? new Array(4).fill(this.edgeSpace[0])
        : []
      : [];
    this.edgeSpace = [x0, x1, y0, y1];
  },
  computed: {
    // unit handler, if value without unit then add 'px' string at the end
    positions() {
      let p = {};
      let point = {};
      if (this.isTransformSupport) {
        point = this.defaultPoint;
        p["transform"] =
          "translate3d(" +
          this.tCurrentPoint.x +
          "px, " +
          this.tCurrentPoint.y +
          "px, 0)";
      } else {
        point = this.position;
      }
      for (let key in point) {
        const value = point[key];
        if (this.numReg.test(value)) {
          p[key] = value + "px";
        }
      }
      return p;
    }
  },
  methods: {
    getRange(key, type) {
      const [x0 = 0, x1 = 0, y0 = 0, y1 = 0] = this.edgeSpace;
      let result = [];
      if (key === "left" || key === "top") {
        result = [
          -this.position[key] + x0,
          this.screenceSize[this.translateRangeMap[key].type] -
            this.position[key] -
            this.wrapSize[type] -
            x1
        ];
      } else if (key === "right" || key === "bottom") {
        result = [
          -(
            this.screenceSize[this.translateRangeMap[key].type] -
            this.position[key] -
            this.wrapSize[type] +
            y0
          ),
          this.position[key] - y1
        ];
      }
      return result;
    },
    getPlatform() {
      const agents = ["Android", "iPhone", "Windows Phone", "iPad"];
      const userAgent = navigator.userAgent;
      for (let ag of agents) {
        // if match return platform type
        if (userAgent.includes(ag)) {
          return ag;
        }
      }
      // if it isn't mobile return false
      return false;
    },
    cssSupported(property) {
      return property in document.body.style;
    },
    // get current event point
    getPoint($event) {
      const changedTouches = this.platform
        ? $event.changedTouches[0]
        : { clientX: $event.clientX, clientY: $event.clientY };
      return {
        x: Math.ceil(changedTouches.clientX || changedTouches.pageX),
        y: Math.ceil(changedTouches.clientY || changedTouches.pageY)
      };
    },
    // boundary handler function
    boundaryHandler(key) {
      const { type, name, index } = this.translateRangeMap[key];
      const [rMin, rMax] = this.positionRange[type];
      const mv = this.eleMoveDistance(type, index);
      const ms = mv + this.startPoint[name];
      if (ms <= rMin) {
        return rMin;
      } else if (ms >= rMax) {
        return rMax;
      } else {
        return ms;
      }
    },
    // get element mv distance
    eleMoveDistance(type, index) {
      if (this.isTransformSupport) {
        return this.dis[type];
      } else {
        return !index ? this.dis[type] : -this.dis[type];
      }
    },
    start($event) {
      this.startTouchInfo = this.getPoint($event);
      this.$emit("start", this.startTouchInfo);
      this.doc.addEventListener(this.evs.MOVE, this.move);
    },
    move($event) {
      // prevent default behaviour
      $event.preventDefault();
      // get current event point
      const currentTouchInfo = this.getPoint($event);
      // get finger move distance
      this.dis.x = currentTouchInfo.x - this.startTouchInfo.x;
      this.dis.y = currentTouchInfo.y - this.startTouchInfo.y;
      let point = this.isTransformSupport ? this.tCurrentPoint : this.position;
      for (let key in this.position) {
        const type = this.isTransformSupport
          ? this.translateRangeMap[key].type
          : key;
        point[type] = this.boundaryHandler(key);
      }
      this.$emit("move", { ...point }, { ...this.dis });
    },
    end($event) {
      // synchro update exist property from position
      for (let key in this.position) {
        this.startPoint[key] = this.isTransformSupport
          ? this.tCurrentPoint[this.translateRangeMap[key].type]
          : this.position[key];
      }
      this.doc.removeEventListener(this.evs.MOVE, this.move);
      this.$emit("end", { ...this.startPoint });
    }
  },
  mounted() {
    // get component root element
    this.wrap = this.$refs["eMove"];
    // temp element size data
    this.wrapSize = {
      x: this.wrap.clientWidth,
      y: this.wrap.clientHeight
    };
    if (this.isTransformSupport) {
      // get move range
      for (let key in this.position) {
        const { type } = this.translateRangeMap[key];
        this.positionRange[type] = this.getRange(key, type);
      }
    } else {
      // x: [min, max], y: [min, max], available darg area
      this.positionRange = {
        x: [x0, this.screenceSize.x - x1 - this.wrapSize.x],
        y: [y0, this.screenceSize.y - y1 - this.wrapSize.y]
      };
    }
    // add event listener
    this.$refs["eMove"].addEventListener(this.evs.START, this.start);
    this.doc.addEventListener(this.evs.END, this.end);
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.e-move {
  position: fixed;
}
/* 
  1、add drag available area, via edge space array attribute
  2、add [start move、move、 end] event
   */
</style>


