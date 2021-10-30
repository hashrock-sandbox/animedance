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
    </div>

    <!-- list frames -->
    <div class="frames">
      <div v-for="(frame, index) in frames" :key="index"
        :class="{ active: index === currentFrame }"
        @click="currentFrame = index"
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
        
        // const offscreenCanvas = offscreenCanvases[frame.uuid];
        // if(offscreenCanvas){
        //   ctx.drawImage(offscreenCanvas, 0, 0);
        // }
      }
    },
    insertFrame(){
      const offscreenCanvas = document.createElement('canvas');
      offscreenCanvas.width = this.width;
      offscreenCanvas.height = this.height;
      const ctx = offscreenCanvases[uuid()] = offscreenCanvas.getContext('2d');
      ctx.drawImage(this.$refs.canvas, 0, 0);
      this.frames.splice(this.currentFrame, 0, {
        uuid: uuid(),
        data: offscreenCanvas.toDataURL()
      });
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
.thumb{
  width: 100px;
  height: 100px;
  border: 1px solid black;
}
.canvas{
  border: 1px solid black;
}
</style>