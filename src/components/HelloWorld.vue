<template>
  <canvas ref="can" width="640" height="480"></canvas>
  <button @click="listaDados">Lista Objetos</button>
  <button @click="adicionaPoligono">Novo Poligono</button>
  <button @click="adicionaQuadrado">Novo Quadrado</button>
  <button @click="editaObjeto">Edita</button>
  <button @click="paraEdicao">Para Edicao</button>
  <button >Destaca</button>
  <button >Oculta/Desoculta</button>
  <button >Apaga</button>
  <input v-model="message"/>
</template>

<script>
import { fabric } from 'fabric';

export default {
  data(){
    return {
      canvas: null,
      message:"0",
      desenhando: {
        poligono: null,
        linha: null,
        linhas: [],
        pontos: []
      },
      objetos: [],
      ultimaRef: -1,
      edicaoCirculos: [],
      modo: 0, //0 - nada, 1 - segmentacao, 2 - bbox
    }
  },
  mounted() {
    const ref = this.$refs.can;
    this.canvas = new fabric.Canvas(ref);
    this.canvas.selection = false;
    this.canvas.on('mouse:wheel', this.zoomScroll);
    this.canvas.on('object:moving', this.moveObject);
    this.canvas.on('mouse:down', this.mouseDown);
    this.canvas.on('mouse:up', this.mouseUp);
    this.canvas.on('mouse:move', this.mouseMove);
    this.inicia("https://media.discordapp.net/attachments/905770077251600396/1040329314241101904/black_640x480.png");
  },
  
  methods: {
    zoomScroll(opt){
      let delta = opt.e.deltaY;
      let zoom = this.canvas.getZoom();
      zoom *= 0.999 ** delta;
      if (zoom > 15) zoom = 15;
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
      if(this.desenhando.linha && this.desenhando.linha.class == "line" && this.modo == 1){
        this.desenhando.linha.set({ x2: event.absolutePointer.x, y2: event.absolutePointer.y});
        this.canvas.renderAll();
        //desenha rastro linha
      }
      else if (this.modo == 2 && this.desenhando.poligono){
        this.desenhando.poligono.set({ width: (this.desenhando.poligono.left - event.absolutePointer.x) *-1 });
        this.desenhando.poligono.set({ height: (this.desenhando.poligono.top - event.absolutePointer.y) *-1});
        this.canvas.renderAll();
      }
    },
    mouseDown(event){
      if(event.target && this.desenhando.pontos[0].id == event.target.id){
        this.desenhaPoligono();
      }
      else if(this.modo == 1){
        this.adicionaPonto(event);
      }
      else if( this.modo == 2){
        this.comecaQuadrado(event);
      }
    },
    mouseUp(event){
      if(this.modo == 2){
        this.fechaQuadrado(event);
      }
    },
    //PRONTO
    adicionaPonto(event){
      let pontoAtual = event.absolutePointer;

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

    comecaQuadrado(event){
      let pontoAtual = event.absolutePointer;

      //adiciona um circulo na posicao clicada
      //gero um id unico pro ponto
      var random = Math.floor(Math.random() * (999999 - 99 + 1)) + 99;
      var id = new Date().getTime() + random;
      
      var circle = new fabric.Circle({
        radius: 3,
        fill: 'red',
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
      this.desenhando.pontos.push(circle);
      this.canvas.add(circle);


      this.ultimaRef++;
      var rect = new fabric.Rect({
        left: pontoAtual.x,
        top: pontoAtual.y,
        stroke: '#333333',
        strokeWidth: 0.5,
        fill: 'white',
        opacity: 0.2,
        selectable: false,
            objectCaching: false,
      });

      this.desenhando.poligono = rect;
      this.canvas.add(rect);
      
    },

    fechaQuadrado(event){
      let pontoAtual = event.absolutePointer;
      this.desenhando.poligono.set({ width: (this.desenhando.poligono.left - pontoAtual.x)*-1  });
      this.desenhando.poligono.set({ height: (this.desenhando.poligono.top - pontoAtual.y)*-1  });
      this.canvas.renderAll();
      this.canvas.remove(this.desenhando.pontos[0]);
      this.objetos.push(this.desenhando.poligono);
      this.modo = 0
    },

    //VERIFICAR
    inicia(image){
      this.canvas.setBackgroundColor({
        source: image,
        offsetX: 640,
        offsetY: 480
      }, 
      this.canvas.renderAll.bind( this.canvas));
    },
    //DEBUG
    listaDados(){
      this.canvas.setZoom(1);
      console.log("lista poligonos",this.objetos);
      console.log("ultimaRef",this.ultimaRef);
      console.log("zoom",this.canvas.getZoom());
    },
    adicionaPoligono(){
      this.desenhando = {
        poligono: null,
        linha: null,
        linhas: [],
        pontos: []
      };
      this.modo = 1;
    },
    adicionaQuadrado(){
      this.desenhando = {
        poligono: null,
        linha: null,
        linhas: [],
        pontos: []
      };
      this.modo = 2;
    },
    editaObjeto(){
      this.edicaoCirculos=[];
      let objeto = this.objetos[parseInt(this.message)]
      if (objeto.type == "polygon"){
        objeto.points.forEach((element, index) => {
          var circle = new fabric.Circle({
            radius: 3,
            fill: '#ffffff',
            stroke: '#333333',
            strokeWidth: 0.5,
            left: (this.desenhando.poligono.left),
            top: (this.desenhando.poligono.top),
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
      }
      else if(objeto.type == "rect"){
        var circleIni = new fabric.Circle({
          radius: 3,
          fill: '#ffffff',
          stroke: '#333333',
          strokeWidth: 0.5,
          left: (this.desenhando.poligono.left),
          top: (this.desenhando.poligono.top),
          hasBorders: false,
          hasControls: false,
          originX: 'center',
          originY: 'center',
          id: 0,
          objectCaching: false
        });
        this.canvas.add(circleIni);
        this.edicaoCirculos.push(circleIni);
        var circleFim = new fabric.Circle({
          radius: 3,
          fill: '#ffffff',
          stroke: '#333333',
          strokeWidth: 0.5,
          left: (this.desenhando.poligono.left + this.desenhando.poligono.width),
          top: (this.desenhando.poligono.top + this.desenhando.poligono.height),
          hasBorders: false,
          hasControls: false,
          originX: 'center',
          originY: 'center',
          id: 0,
          objectCaching: false
        });
        this.canvas.add(circleFim);
        this.edicaoCirculos.push(circleFim);        
      }
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



