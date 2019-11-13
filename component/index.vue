<template>
  <div class="e-move" ref="eMove" :style="positions">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "VueDragElement",
  props:{
    options: {
     type: Object,
     default: {}
    }
  },
  data() {
    return {
      doc: document,
      wrap: null,
      evs: {},
      initPoint: { ...this.options.position },
      touchInfo:{
        start:{}
      },
      dis:{ x:0, y:0 },
      numReg: /^\d+$/,
      platform: this.getPlatform(),
      wrapSize: { width: 0, height: 0 },
      screenSize:{ width: 0, height: 0 },
      // DIRCCTION_X as x director
      DIRCCTION_X : 0,
      // DIRCCTION_Y as y director
      DIRCCTION_Y : 1,
      directionMap: {},
      // handleTypes attribute
      handleTypes: [
          { 
            left: 0,
            right: 1,
            sizeType : 'width',
            disType:'x'
           },
           { 
            top: 0,
            bottom: 1,
            sizeType : 'height',
            disType:'y'
           }
        ],
        position: {  ...this.options.position }
    };
  },
  created() {
    this.evs = {
        START: this.platform ? 'touchstart' : 'mousedown',
        MOVE: this.platform ? 'touchmove' : 'mousemove',
        END: this.platform ? 'touchend' : 'mouseup',
      }
    this.directionMap =  {
        left: this.DIRCCTION_X ,
        right: this.DIRCCTION_X ,
        top: this.DIRCCTION_Y ,
        bottom: this.DIRCCTION_Y 
    }
    // get screen size
    const sc =  window.screen
    this.screenSize = {
      width: document.body.clientWidth || document.documentElement.clientWidth,
      height: document.body.clientHeight || document.documentElement.clientHeight,
    }
  },
  computed:{
    // unit handler, if value without unit then add 'px' string at the end
    positions (){
      let p = {};
      for(let key in this.position){
        const value = this.position[key]
        if(this.numReg.test(value)){
          p[key] = value + 'px'
        }
      }
      return p;
    }
  },
  methods: {
    // add default value, if needed
    getPlatform(){
      const agents = ["Android", "iPhone", "Windows Phone", "iPad"]
      const userAgent = navigator.userAgent
      for(let ag of agents){
        // if match return platform type
        if(userAgent.includes(ag)){
          return ag
        }
      }
      // if it isn't mobile return false
      return false;
    },
    // get current event point
    getPoint($event){
      const changedTouches = this.platform ? $event.changedTouches[0] : { clientX: $event.clientX, clientY: $event.clientY }
       return {
        pX: Math.ceil(changedTouches.clientX || changedTouches.pageX),
        pY: Math.ceil(changedTouches.clientY || changedTouches.pageY)
      }
    },
    // boundary handler function
    // director  0 horizontal ，1 verticle
    boundaryHandler(type, director){
        // get handler type (horizontal or vertical)
        const handleType  = this.handleTypes[director]
        const { disType, sizeType } = handleType;
        const mv = !handleType[type] ?
        this.initPoint[type] + this.dis[disType] :  this.initPoint[type] - this.dis[disType];
        if( mv <= 0 ){
          this.position[type] = 0
        }else if( mv + this.wrapSize[sizeType] >= this.screenSize[sizeType] ){
          this.position[type] = this.screenSize[sizeType] - this.wrapSize[sizeType]
        }else{
          this.position[type] = mv
        }
    },
    start($event){
      this.touchInfo[this.evs.START] = this.getPoint($event)
      // if it isn't mobile
      this.doc.addEventListener(this.evs.MOVE, this.move)
    },
    move($event){
      // prevent default behaviour
      $event.preventDefault();
      // get current event point
      const currentTouchInfo = this.getPoint($event)
      // get finger move distance
      this.dis.x = currentTouchInfo.pX - this.touchInfo[this.evs.START].pX;
      this.dis.y = currentTouchInfo.pY - this.touchInfo[this.evs.START].pY;
      for(let key in this.position){
        this.boundaryHandler(key, this.directionMap[key])
      }
    },
    end($event){
      // synchro update exist property from position
      for(let key in this.position){
        this.initPoint[key] = this.position[key]
      }
      // if it isn't mobile
      this.doc.removeEventListener(this.evs.MOVE, this.move)
    }
  },
  mounted(){
    // get component root element
    this.wrap = this.$refs['eMove'];
    // temp element size data
    this.wrapSize = {
      width: this.wrap.clientWidth,
      height: this.wrap.clientHeight
    }
    // add event listener
    this.$refs['eMove'].addEventListener(this.evs.START, this.start)
    // if mobile
    this.doc.addEventListener(this.evs.END, this.end)
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .e-move{
    position: fixed;
  }
</style>
