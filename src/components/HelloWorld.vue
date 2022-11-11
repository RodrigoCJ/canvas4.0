<template>
  <canvas width="640" height="480" id="canvas" ></canvas>
  <button @click="listaDados">Lista Objetos</button>
  <button @click="adicionaPoligono">Novo Poligono</button>
  <button @click="adicionaQuadrado">Novo Quadrado</button>
  <button @click="editaObjeto">Edita</button>
  <button @click="paraEdicao">Para Edicao</button>
  <button @click="destaca">Destaca</button>
  <button @click="showHide">Oculta/Desoculta</button>
  <button @click="apaga">Apaga</button>
  <button @click="cancela">Cancela</button>
  <button @click="desfaz">Desfaz</button>
  <input v-model="message" placeholder="id objeto"/>
  <p>
    Linha
    <input type="color" id="head" name="head" v-model="parametros.corBorda">
    <input v-model="parametros.grosuraBorda" placeholder="grosura borda"/>
  </p>
  <p>
    Preenchimento/Ponto
    <input type="color" id="head" name="head" v-model="parametros.corPreenchimento">
    <input v-model="parametros.transparenciaPreenchimento" placeholder="transparencia preenchimento entre 0 e 1"/>
  </p>
  <p>
    <input type="checkbox" v-model="parametros.mostraID"/>
    Exibir ID
  </p>
  <button @click="atualizar">Atualizar</button>
  

</template>

<script>
import { fabric } from 'fabric';

export default {
  data(){
    return {
      canvas: null,
      message:"0",
      desenhando: {
        poligono: null, //objeto que esta sendo desenhado
        linha: null,   //linha que esta sendo desenhada atualmente
        linhas: [],   //lista de linhas que foram desenhadas no canvas entre os pontos
        pontos: []    //lista circulos desenhados no canvas nos pontos em que foram clicados
      },
      parametros:{
        mostraID: true,
        corBorda: "#999999",
        corPreenchimento: "#ffffff",
        grosuraBorda: 1,
        transparenciaPreenchimento: 0.1,
      },
      objetos: [],
      textos: [],
      ultimaRef: -1,
      edicaoCirculos: [],
      imgWidth: 640,
      imgHeight: 480,
      modo: 0, //0 - nada, 1 - segmentacao, 2 - bbox
    }
  },
  mounted() {
    this.canvas = new fabric.Canvas('canvas');
    this.canvas.selection = false;
    this.canvas.on('mouse:wheel', this.zoomScroll);
    this.canvas.on('object:moving', this.moveObject);
    this.canvas.on('mouse:down', this.mouseDown);
    this.canvas.on('mouse:up', this.mouseUp);
    this.canvas.on('mouse:move', this.mouseMove);
    document.addEventListener("keyup", this.tecladoEvent);
    this.inicia("https://media.discordapp.net/attachments/905770077251600396/1040581886331863060/black_640x480.png");
    // this.inicia("https://media.discordapp.net/attachments/947876906185924648/1040255879435526204/7007_1667849369020.jpg");
  },
  
  methods: {
    zoomScroll(event){
      let delta = event.e.deltaY *-1;
      let zoom = this.canvas.getZoom();
      // zoom *= 0.999 ** delta;
      zoom = zoom + delta/200;
      if (zoom > 15) zoom = 15;
      if (zoom < 1) zoom = 1;
      this.canvas.zoomToPoint({ x: event.e.offsetX, y: event.e.offsetY }, zoom);
      event.e.preventDefault();
      event.e.stopPropagation();
      var vpt = this.canvas.viewportTransform;
      if (vpt[4] >= 0) {
        this.canvas.viewportTransform[4] = 0;
      } 
      else if (vpt[4] < this.canvas.getWidth() - this.imgWidth * zoom) {
        this.canvas.viewportTransform[4] = this.canvas.getWidth() - this.imgWidth * zoom;
      }
      if (vpt[5] >= 0) {
        this.canvas.viewportTransform[5] = 0;
      } 
      else if (vpt[5] < this.canvas.getHeight() - this.imgHeight * zoom) {
        this.canvas.viewportTransform[5] = this.canvas.getHeight() - this.imgHeight * zoom;
      }
    },
    moveObject(event){
      let a = parseInt(this.message);
      let objs = this.objetos.filter(function(obj){
        return obj.id == a;
      });
      let objeto = objs[0];
      var p = event.target;
      if(objeto.type == "polygon"){
        objeto.points[p.id] = {x: p.getCenterPoint().x, y: p.getCenterPoint().y};
      }
      else if(objeto.type == "rect"){
       let cordIni = {x:objeto.left, y: objeto.top};
       let cordFin = {x:objeto.left + objeto.width, y: objeto.top + objeto.height};

        if(p.id == 'i'){
          cordIni.x = p.getCenterPoint().x;
          cordIni.y = p.getCenterPoint().y;
        }
        if(p.id == 'f'){
          cordFin.x = p.getCenterPoint().x;
          cordFin.y = p.getCenterPoint().y;
        }

       //inicial
       objeto.set({ left: (cordIni.x)  });
       objeto.set({ top: (cordIni.y)  });

       //final
       objeto.set({ width: (cordIni.x - cordFin.x)*-1  });
       objeto.set({ height: (cordIni.y - cordFin.y)*-1  });
      }   
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
      if(event.target && this.desenhando.pontos.length > 0 && this.desenhando.pontos[0].id == event.target.id){
        this.desenhaPoligono();
      }
      else if(this.modo == 1){
        this.adicionaPonto(event.absolutePointer);
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
    adicionaPonto(pontoAtual){

      //adiciona um circulo na posicao clicada
      //gero um id unico pro ponto
      var random = Math.floor(Math.random() * (999999 - 99 + 1)) + 99;
      var id = new Date().getTime() + random;
      
      var circle = new fabric.Circle({
        radius: 3,
        fill: this.parametros.corPreenchimento,
        stroke: this.parametros.corBorda,
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
            fill: this.parametros.corPreenchimento,
            stroke: this.parametros.corBorda,
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
            stroke: this.parametros.corBorda,
            strokeWidth: 0.5,
            fill: this.parametros.corPreenchimento,
            opacity: 0.2,
            hasBorders: false,
            hasControls: false,
            id: this.ultimaRef,
            selectable: false,
            objectCaching: false,
        });
      this.canvas.add(polygon);
      this.objetos.push(polygon);
      this.mostraID();
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
        fill: this.parametros.corPreenchimento,
        stroke: this.parametros.corBorda,
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
        stroke: this.parametros.corBorda,
        strokeWidth: 0.5,
        fill: this.parametros.corPreenchimento,
        id: this.ultimaRef,
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
      this.mostraID();
      this.modo = 0
    },
    tecladoEvent(event) {
      //ctrl + z
      if (event.ctrlKey && event.key === "z") {
        this.desfaz();
      }
      if (event.keyCode === 27) {
        this.cancela();
      }
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
      var vpt = this.canvas.viewportTransform;
      if (vpt[4] >= 0) {
        this.canvas.viewportTransform[4] = 0;
      } 
      else if (vpt[4] < this.canvas.getWidth() - this.imgWidth * 1) {
        this.canvas.viewportTransform[4] = this.canvas.getWidth() - this.imgWidth * 1;
      }
      if (vpt[5] >= 0) {
        this.canvas.viewportTransform[5] = 0;
      } 
      else if (vpt[5] < this.canvas.getHeight() - this.imgHeight * 1) {
        this.canvas.viewportTransform[5] = this.canvas.getHeight() - this.imgHeight * 1;
      }
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
      let a = parseInt(this.message);
      let objs = this.objetos.filter(function(obj){
        return obj.id == a;
      });
      let objeto = objs[0]
      if (objeto.type == "polygon"){
        objeto.points.forEach((element, index) => {
          var circle = new fabric.Circle({
            radius: 3,
            fill: this.parametros.corPreenchimento,
            stroke: this.parametros.corBorda,
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
      }
      else if(objeto.type == "rect"){
        var circleIni = new fabric.Circle({
          radius: 3,
          fill: this.parametros.corPreenchimento,
          stroke: this.parametros.corBorda,
          strokeWidth: 0.5,
          left: (objeto.left),
          top: (objeto.top),
          hasBorders: false,
          hasControls: false,
          originX: 'center',
          originY: 'center',
          id: 'i',
          objectCaching: false
        });
        this.canvas.add(circleIni);
        this.edicaoCirculos.push(circleIni);
        var circleFim = new fabric.Circle({
          radius: 3,
          fill: this.parametros.corPreenchimento,
          stroke: this.parametros.corBorda,
          strokeWidth: 0.5,
          left: (objeto.left + objeto.width),
          top: (objeto.top + objeto.height),
          hasBorders: false,
          hasControls: false,
          originX: 'center',
          originY: 'center',
          id: 'f',
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
    },
    apaga(){
      let forma = parseInt(this.message);
      let remover = this.objetos.filter(function(obj){
        return obj.id == forma;
      });
      this.canvas.remove(remover[0])
      this.objetos = this.objetos.filter(function(obj){
        return obj.id != forma;
      })
    },
    showHide(){
      this.objetos.forEach((object,index) => {
        object.visible = !object.visible;
      });
      this.canvas.renderAll();
    },
    destaca(){
      let a = parseInt(this.message)
      let objs = this.objetos.filter(function(obj){
        return obj.id == a;
      });
      let objeto = objs[0]
      objeto.fill= '#ffffff';
      objeto.stroke= '#333333';
      objeto.visible = true;
      this.canvas.renderAll();
    },
    cancela(){
      if(this.modo != 0){
        this.desenhando.pontos.forEach((point,index) => {
        this.canvas.remove(point);
      });
      //apago as linhas desenhadas
      this.desenhando.linhas.forEach((linha,index) => {
        this.canvas.remove(linha);
      })
      this.canvas.remove(this.desenhando.poligono)
      this.modo = 0;
      }
    },
    desfaz(){
      if(this.modo==1){
        if(this.desenhando.pontos.length >1){
          //apago o ultimo ponto
          this.canvas.remove(this.desenhando.pontos[this.desenhando.pontos.length-1]);
          this.desenhando.pontos.pop();
          this.canvas.remove(this.desenhando.pontos[this.desenhando.pontos.length-1]);
          this.desenhando.pontos.pop();
          //apago a ultima linha
          this.canvas.remove(this.desenhando.linhas[this.desenhando.linhas.length-1]);
          this.desenhando.linhas.pop();
          this.canvas.remove(this.desenhando.linhas[this.desenhando.linhas.length-1]);
          let removido = this.desenhando.linhas.pop();
          this.adicionaPonto({x: removido.x1, y:removido.y1})
        }
        else{
          this.cancela()
          this.adicionaPoligono()
        }
      }
      else if (this.modo ==2 ){
        this.cancela()
        this.adicionaQuadrado()
      }
      
    },
    mostraID(){
      if(this.parametros.mostraID){
        this.objetos.forEach((obj,index) => {
          console.log("obj id",obj.id)
          if(obj.type == "polygon"){
            var text = new fabric.Text(""+obj.id,{
              left: obj.points[0].x,
              top:obj.points[0].y,
              id: obj.id,
              fill: this.parametros.corBorda,
            });
            this.textos.push(text);
            this.canvas.add(text);
          }
          else if(obj.type == "rect"){
            var text1 = new fabric.Text(""+obj.id,{
              left: obj.left,
              top:obj.top,
              id: obj.id,
              fill: this.parametros.corBorda,
            });
            this.textos.push(text1);
            this.canvas.add(text1);
          }
        }); 
      }
    },
  }
}
</script>



