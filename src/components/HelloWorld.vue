<template>
  <canvas ref="can" width="600" height="600"></canvas>
</template>

<script>
import { fabric } from 'fabric';

export default {
  data(){
    return {
      canvas: null
    }
  },
  mounted() {
    const ref = this.$refs.can;
    this.canvas = new fabric.Canvas(ref);
    this.canvas.on('mouse:wheel', function(opt) {
      console.log("werqwer")
      var delta = opt.e.deltaY;
      var zoom = this.canvas.getZoom();
      zoom *= 0.999 ** delta;
      if (zoom > 20) zoom = 20;
      if (zoom < 0.01) zoom = 0.01;
      this.canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
      opt.e.preventDefault();
      opt.e.stopPropagation();
    });
    this.inicia();
  },
  methods: {



    inicia(image){
      this.canvas.setBackgroundColor({
        source: 'http://fabricjs.com/assets/escheresque_ste.png',
        repeat: 'repeat',
        offsetX: 600,
        offsetY: 600
      }, this.canvas.renderAll.bind( this.canvas));
    }
  },
};
</script>
