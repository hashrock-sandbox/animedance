<template>
  <div>
    <canvas ref="canvas" class="canvas" :width="width" :height="height"
      @pointerdown="onPointerDown" @pointermove="onPointerMove"
      @pointerup="onPointerUp"
      ></canvas>
    <div class="controls">
      <div>
        {{currentFrame}}
      </div>

      <button @click="prev">Prev</button>
      <button @click="next">Next</button>
      <button @click="insertFrame">Insert Frame</button>
      <button @click="insertAfter">Insert After</button>
      <button @click="deleteFrame">Delete Frame</button>

    </div>

    <!-- list frames -->
    <div class="frames">
      <div v-for="(frame, index) in frames" :key="index"
        class="frame-item"
        :class="{ active: index === currentFrame }"
        @click="loadFrame(index)"
      >
        {{ index }}
        <img :src="frame.data" class="thumb" />
      </div>
    </div>
  </div>
</template>

<script>
const offscreenCanvases = {};

function uuid(){
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
    var r = Math.random()*16|0, v = c == 'x' ? r : (r&0x3|0x8);
    return v.toString(16);
  });
}


export default {
  data(){
    return {
      currentFrame : 0,
      frames: [],
      ctx: null
    }
  },
  props: {
    width: {
      type: Number,
      default: 500
    },
    height: {
      type: Number,
      default: 500
    }
  },
  methods: {
    onPointerDown(e){
      this.isDrawing = true;
      this.lastX = e.offsetX;
      this.lastY = e.offsetY;
    },
    onPointerMove(e){
      if(!this.isDrawing) return;
      const ctx = this.ctx;
      const x = e.offsetX;
      const y = e.offsetY;
      ctx.beginPath();
      ctx.moveTo(this.lastX, this.lastY);
      ctx.lineTo(x, y);
      ctx.stroke();
      this.lastX = x;
      this.lastY = y;
    },
    onPointerUp(e){
      this.isDrawing = false;
      // update thumb
      const ctx = this.ctx;
      const canvas = ctx.canvas;
      const data = canvas.toDataURL();
      const frame = {
        data: data,
        id: uuid()
      };
      this.frames[this.currentFrame] = frame;
      // this.frames.push(frame);
    },
    loadFrame(index){
      const frame = this.frames[index];
      if(!frame) return;
      const ctx = this.ctx;
      const canvas = ctx.canvas;
      const img = new Image();
      img.src = frame.data;
      img.onload = () => {
        ctx.drawImage(img, 0, 0);
      }
      this.currentFrame = index;
    },
    prev(){
      this.currentFrame--;
      this.draw();
    },
    next(){
      this.currentFrame++;
      this.draw();
    },
    draw(){
      const ctx = this.$refs.canvas.getContext('2d');
      ctx.fillStyle = '#fff';
      ctx.fillRect(0, 0, this.width, this.height);

      if(this.frames[this.currentFrame]){
        const frame = this.frames[this.currentFrame];
        const img = new Image();
        img.src = frame.data;
        img.onload = () => {
          ctx.drawImage(img, 0, 0);
        }
      }
    },
    crearCanvas(){
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext('2d');
      ctx.fillStyle = '#fff';
      ctx.fillRect(0, 0, this.width, this.height);
    },
    insertFrame(){
      const offscreenCanvas = document.createElement('canvas');
      offscreenCanvas.width = this.width;
      offscreenCanvas.height = this.height;

      const ctx = offscreenCanvas.getContext('2d');
      this.crearCanvas()
      // const ctx = offscreenCanvases[uuid()] = offscreenCanvas.getContext('2d');
      // ctx.drawImage(this.$refs.canvas, 0, 0);
      this.frames.splice(this.currentFrame, 0, {
        uuid: uuid(),
        data: offscreenCanvas.toDataURL()
      });
    },
    insertAfter(){
      const offscreenCanvas = document.createElement('canvas');
      offscreenCanvas.width = this.width;
      offscreenCanvas.height = this.height;

      const ctx = offscreenCanvas.getContext('2d');
      this.crearCanvas()
      // const ctx = offscreenCanvases[uuid()] = offscreenCanvas.getContext('2d');
      // ctx.drawImage(this.$refs.canvas, 0, 0);
      this.frames.splice(this.currentFrame + 1, 0, {
        uuid: uuid(),
        data: offscreenCanvas.toDataURL()
      });
      this.currentFrame++;
    },
    deleteFrame(){
      this.frames.splice(this.currentFrame, 1);
      this.currentFrame = 0;
      this.draw();
    }
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext('2d')
    this.draw()

    // this.insertFrame()

  },

}
</script>

<style>
/* .thumb{
  width: 100px;
  height: 100px;
  border: 1px solid black;
  display: inline-block;
} */
.frame-item{
  display: inline-block;
  width: 100px;
  height: 100px;
  border: 1px solid black;
  margin: 5px;
  cursor: pointer;
}
.frame-item.active{
  border: 1px solid red;
}
.frame-item img{
  width: 100%;
  height: 100%;
}
.canvas{
  border: 1px solid black;
}
</style>