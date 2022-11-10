<template>
  <canvas ref="can" width="600" height="600"></canvas>
  <button @click="listaPoligonos">Lista Poligonos</button>
  <button @click="reiniciaPoligono">Reinicia Poligono</button>
  <button @click="EditaPoligono">Edita</button>
  <button @click="paraEdicao">Para Edicao</button>
</template>

<script>
import { fabric } from 'fabric';

export default {
  data(){
    return {
      canvas: null,
      desenhando: {
        poligono: null,
        linha: null,
        linhas: [],
        pontos: []
      },
      objetos: [],
      ultimaRef: -1,
      edicaoCirculos: [],
      modo: 1, //0 - nada, 1 - segmentacao, 2 - bbox
    }
  },
  mounted() {
    const ref = this.$refs.can;
    this.canvas = new fabric.Canvas(ref);
    this.canvas.on('mouse:wheel', this.zoomScroll);
    this.canvas.on('object:moving', this.moveObject);
    this.canvas.on('mouse:down', this.mouseDown);
    this.canvas.on('mouse:move', this.mouseMove);
    this.inicia("http://fabricjs.com/assets/escheresque_ste.png");
  },
  
  methods: {
    zoomScroll(opt){
      let delta = opt.e.deltaY;
      let zoom = this.canvas.getZoom();
      zoom *= 0.999 ** delta;
      if (zoom > 20) zoom = 20;
      if (zoom < 1) zoom = 1;
      this.canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
      opt.e.preventDefault();
      opt.e.stopPropagation();
    },
    moveObject(event){
      var p = event.target;
      this.objetos[0].points[p.id] = {x: p.getCenterPoint().x, y: p.getCenterPoint().y};
    },
    mouseMove(event){
      if(this.desenhando.linha && this.desenhando.linha.class == "line"){
        this.desenhando.linha.set({ x2: event.absolutePointer.x, y2: event.absolutePointer.y});
        this.canvas.renderAll();
        //desenha rastro linha
      }
    },
    mouseDown(event){
      if(event.target && this.desenhando.pontos[0].id == event.target.id){
        this.desenhaPoligono();
      }
      else if(this.modo == 1){
        this.adicionaPonto(event);
      }
    },
    //PRONTO
    adicionaPonto(event){
      let pontoAtual = event.absolutePointer;
      console.log("get point",pontoAtual)

      //adiciona um circulo na posicao clicada
      //gero um id unico pro ponto
      var random = Math.floor(Math.random() * (999999 - 99 + 1)) + 99;
      var id = new Date().getTime() + random;
      
      var circle = new fabric.Circle({
        radius: 3,
        fill: '#ffffff',
        stroke: '#333333',
        strokeWidth: 0.5,
        left: (pontoAtual.x),
        top: (pontoAtual.y),
        selectable: false,
        hasBorders: false,
        hasControls: false,
        originX: 'center',
        originY: 'center',
        id: id,
        objectCaching: false
      });
      //verifico se e o primeiro ponto, e se for mudo a cor
      if (this.desenhando.pontos.length == 0) {
        circle.set({
            fill: 'red'
        })
      }
      this.desenhando.pontos.push(circle);
      this.canvas.add(circle);

      //cria a linha de rastro
      var points = [(pontoAtual.x), (pontoAtual.y), (pontoAtual.x), (pontoAtual.y)];
      let line = new fabric.Line(points, {
            strokeWidth: 2,
            fill: '#999999',
            stroke: '#999999',
            class: 'line',
            originX: 'center',
            originY: 'center',
            selectable: false,
            hasBorders: false,
            hasControls: false,
            evented: false,
            objectCaching: false
        });
      this.desenhando.linha = line;
      this.desenhando.linhas.push(line);
      this.canvas.add(line);
    },
    desenhaPoligono(){
      var pontosTemp = [];

      //pego uma lista de coordenadas dos pontos, e apago os circulos
      this.desenhando.pontos.forEach((point,index) => {
        pontosTemp.push({
          x: point.left,
          y: point.top
        });
        this.canvas.remove(point);
      });
      //apago as linhas desenhadas
      this.desenhando.linhas.forEach((linha,index) => {
        this.canvas.remove(linha);
      })
      //desenho o poligo final
      this.ultimaRef++;
      var polygon = new fabric.Polygon(pontosTemp, {
            stroke: '#333333',
            strokeWidth: 0.5,
            fill: 'red',
            opacity: 0.1,
            hasBorders: false,
            hasControls: false,
            id: this.ultimaRef,
            selectable: false,
            objectCaching: false,
        });
      this.canvas.add(polygon);
      this.objetos.push(polygon);
      this.modo=0;
    },
    //VERIFICAR
    inicia(image){
      this.canvas.setBackgroundColor({
        source: image,
        repeat: 'repeat',
        offsetX: 600,
        offsetY: 600
      }, 
      this.canvas.renderAll.bind( this.canvas));
    },
    //DEBUG
    listaPoligonos(){
      console.log("lista poligonos",this.objetos);
      console.log("ultimaRef",this.ultimaRef);
      console.log("zoom",this.canvas.getZoom());
    },
    reiniciaPoligono(){
      this.desenhando = {
        poligono: null,
        linha: null,
        linhas: [],
        pontos: []
      };
      this.modo = 1;
    },
    EditaPoligono(){
      this.objetos[0].points.forEach((element, index) => {
        var circle = new fabric.Circle({
          radius: 3,
          fill: '#ffffff',
          stroke: '#333333',
          strokeWidth: 0.5,
          left: (element.x),
          top: (element.y),
          hasBorders: false,
          hasControls: false,
          originX: 'center',
          originY: 'center',
          id: index,
          objectCaching: false
        });
        this.canvas.add(circle);
        this.edicaoCirculos.push(circle);
      });
    },
    paraEdicao(){
      this.edicaoCirculos.forEach((point,index) => {
        this.canvas.remove(point);
      });
      this.edicaoCirculos = []
    }

  },
};


</script>



