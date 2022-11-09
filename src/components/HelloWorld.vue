<template>
  <canvas ref="can" width="600" height="600"></canvas>
</template>

<script>
import { fabric } from 'fabric';

export default {
  data(){
    return {
      canvas: null,
      desenhandoLinha: false,
      desenhando: null,
      desenhandoPoints: [],
      objetos: [],
      ultimaRef: -1,
      modo: 1, //0 - nada, 1 - segmentacao, 2 - bbox
    }
  },
  mounted() {
    const ref = this.$refs.can;
    this.canvas = new fabric.Canvas(ref);
    this.canvas.on('mouse:wheel', this.zoomScroll);
    this.canvas.on('object:moving', this.moveMove);
    this.canvas.on('mouse:down', this.mouseDown);
    this.canvas.on('mouse:move', this.mouseMove);
    this.inicia("http://fabricjs.com/assets/escheresque_ste.png");
    // this.poligono();
  },
  
  methods: {
    zoomScroll(opt){
      console.log("zoomScroll chamado");
      let delta = opt.e.deltaY;
      let zoom = this.canvas.getZoom();
      zoom *= 0.999 ** delta;
      if (zoom > 20) zoom = 20;
      if (zoom < 0.01) zoom = 0.01;
      this.canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
      opt.e.preventDefault();
      opt.e.stopPropagation();
    },
    moveMove(event){
      console.log("moveobject chamado");
    },
    mouseMove(event) {
      if(this.desenhandoLinha){
        let pontoAtual = event.absolutePointer;
      }
    },
    mouseDown(event){
      console.log("mouseDown chamado");
      let pontoAtual = event.absolutePointer;
      if(this.modo == 1){
        this.desenhandoPoints.push(pontoAtual);
        let circle = new fabric.Circle({
          radius: 5,
          fill: '#ffffff',
          stroke: '#333333',
          strokeWidth: 0.5,
          left: (pontoAtual.x/this.canvas.getZoom()),
          top: (pontoAtual.y/this.canvas.getZoom()),
          selectable: false,
          hasBorders: false,
          hasControls: false,
          originX:'center',
          originY:'center',
          id:"er",
          objectCaching:false
        });
        this.canvas.add(circle);


        if (this.desenhandoPoints.length != 1){
          this.desenhandoLinha = true;
          if (this.desenhando == null){
            this.desenhando = new fabric.Polygon(this.desenhandoPoints, {
            fill: '#D81B60',
            strokeWidth: 1,
            stroke: 'green',
            id: "!erwq",
            objectCaching: false,
            selectable: false,
            hoverCursor:'dafaul',
            transparentCorners: false,
            });
            this.canvas.add(this.desenhando);
          }
          
        }
      }
    },
    inicia(image){
      this.canvas.setBackgroundColor({
        source: image,
        repeat: 'repeat',
        offsetX: 600,
        offsetY: 600
      }, 
      this.canvas.renderAll.bind( this.canvas));
    },

    poligono(){
      let points = [{x: 3, y: 4}, {x: 16, y: 3}, {x: 30, y: 5},  {x: 25, y: 55}, {x: 19, y: 44}, {x: 15, y: 30}, {x: 15, y: 55}, {x: 9, y: 55}, {x: 6, y: 53}, {x: -2, y: 55}, {x: -4, y: 40}, {x: 0, y: 20}]
      let polygon = new fabric.Polygon(points, {
        fill: '#D81B60',
        strokeWidth: 1,
        stroke: 'green',
        scaleX: 4,
        scaleY: 4,
        id: "!erwq",
        objectCaching: false,
        selectable: false,
        hoverCursor:'dafaul',
        transparentCorners: false,
      });
      this.canvas.add(polygon);
    },
  },
};


</script>



