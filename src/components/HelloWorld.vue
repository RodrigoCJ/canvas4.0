<template>
  <canvas width="640" height="480" id="canvas"></canvas>
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
  <button @click="fullScreen">FullScreen</button>
  <input v-model="message" placeholder="id objeto" />
  <p>
    Linha
    <input type="color" id="head" name="head" v-model="parametros.corLinha" title="Cor da linha"/>
    <input v-model="parametros.grossuraLinha" placeholder="grosura" title="Grossura da linha"/>
    <input v-model="parametros.transparenciaLinha" placeholder="transparencia entre 0 e 1" title="Transparencia da linha"/>
  </p>
  <p>
    Preenchimento
    <input
      type="color"
      id="head"
      name="head"
      v-model="parametros.corPreenchimento"
      title="Cor preenchimento forma"
    />
    <input
      v-model="parametros.transparenciaPreenchimento"
      placeholder="transparencia entre 0 e 1"
      title="Transparencia do preenchimento da forma"
    />
  </p>
  <p>
    Exibir ID
    <input type="checkbox" v-model="parametros.mostraID" />
    <input
      type="color"
      id="head"
      name="head"
      v-model="parametros.corTexto"
      title="Cor do id"
    />
    <input
      v-model="parametros.tamanhoTexto"
      placeholder="tamanho Texto"
      title="Tamnho do id"
    />
  </p>
  <p>
    Esfera
    <input
      type="color"
      id="head"
      name="head"
      v-model="parametros.corEsfera"
      title="Cor das esferas principais"
    />
    <input
      type="color"
      id="head"
      name="head"
      v-model="parametros.corEsferaInter"
      title="Cor das esferas internas"
    />
    <input
      type="color"
      id="head"
      name="head"
      v-model="parametros.corEsferaInicio"
      title="Cor da esfera de inicio"
    />
    <input
      v-model="parametros.raioEsfera"
      placeholder="raioEsfera"
      title="Raio esfera (principal e inicio)"
    />
    <input
      v-model="parametros.raioEsferaInter"
      placeholder="raioEsferaInter"
      title="Raio esfera interna"
    />
    <input
      v-model="parametros.transparenciaEsfera"
      placeholder="transparenciaEsfera"
      title="Transparencia das esferas"
    />
  </p>
  <p>
    Destaque
    <input
      type="color"
      id="head"
      name="head"
      v-model="parametros.corDestaqueForma"
      title="Cor de destaque das formas"
    />
    <input
      type="color"
      id="head"
      name="head"
      v-model="parametros.corDestaqueTexto"
      title="Cor de desataque dos textos"
    />
  </p>
</template>

<script>
import { fabric } from "fabric";

export default {
  data() {
    return {
      canvas: null,
      message: "0",
      desenhando: {
        objeto: null, //objeto que esta sendo desenhado
        linha: null, //linha que esta sendo desenhada atualmente
        linhas: [], //lista de linhas que foram desenhadas no canvas entre os pontos
        pontos: [], //lista circulos desenhados no canvas nos pontos em que foram clicados
      },
      parametros: {
        mostraID: true,
        corLinha: "#ffffff",
        grossuraLinha: 1,
        transparenciaLinha: 0.3,
        corPreenchimento: "#cc3300",
        transparenciaPreenchimento: 0.2,
        corTexto: "#b3b3b3",
        tamanhoTexto: 12,
        corEsfera: "#b3b3b3",
        corEsferaInter: "#0000cc",
        corEsferaInicio: "#ff0000",
        raioEsfera: 3,
        raioEsferaInter: 3,
        transparenciaEsfera: 1,
        corDestaqueForma: "#ffcc00",
        corDestaqueTexto: "#ffffff",
        modo: 0, //0 - nada, 1 - segmentacao, 2 - bbox, 3 - edicao
        fullScreen: false,
        width: 640,
        height: 480,
      },
      objetos: [],    //todos os objetos do fabric
      textos: [],     //todos os textos do fabric
      ultimaRef: -1,  //ultima referencia do objeto
      edicaoCirculos: [], //lista de circulos de edicao no fabric
      pontosIntermendiarios: [], //lista de pontos intermediarios de edicao do fabruic
      destaque: null,   //objeto sendo destacado/editado
      panning: {
        enabled: false,
        moving: false,
        lastX: 0,
        lastY: 0,
      },
    };
  },
  mounted() {
    this.canvas = new fabric.Canvas("canvas");
    this.canvas.selection = false;
    this.canvas.on("mouse:wheel", this.zoomScroll);
    this.canvas.on("object:moving", this.moveObject);
    this.canvas.on("mouse:down", this.mouseDown);
    this.canvas.on("mouse:up", this.mouseUp);
    this.canvas.on("mouse:move", this.mouseMove);
    document.addEventListener("keyup", this.tecladoEventUp);
    document.addEventListener("keydown", this.tecladoEventDown);
    this.inicia("https://media.discordapp.net/attachments/905770077251600396/1040581886331863060/black_640x480.png");
    // this.inicia("https://media.discordapp.net/attachments/947876906185924648/1040255879435526204/7007_1667849369020.jpg");
  },

  methods: {
    //PRONTO
    fullScreen() {
      if (!this.parametros.full) {
        this.canvas.setDimensions(
          {
            width: "1000px",
            height: "840px",
          },
          {
            cssOnly: true,
          }
        );
        this.parametros.full = !this.parametros.full;
      } else {
        this.canvas.setDimensions(
          {
            width: "640px",
            height: "480px",
          },
          {
            cssOnly: true,
          }
        );
        this.parametros.full = !this.parametros.full;
      }
    },

    //PRONTO
    zoomScroll(event) {
      let delta = event.e.deltaY * -1;
      let zoom = this.canvas.getZoom();
      // zoom *= 0.999 ** delta;
      zoom = zoom + delta / 200;
      if (zoom > 15) zoom = 15;
      if (zoom < 1) zoom = 1;
      this.canvas.zoomToPoint({ x: event.e.offsetX, y: event.e.offsetY }, zoom);
      event.e.preventDefault();
      event.e.stopPropagation();
      this.corrigeFoco(zoom);
    },

    //PRONTO
    corrigeFoco(zoom){
      var vpt = this.canvas.viewportTransform;
      if (vpt[4] >= 0) {
        this.canvas.viewportTransform[4] = 0;
      } else if (vpt[4] < this.canvas.getWidth() - this.parametros.width * zoom) {
        this.canvas.viewportTransform[4] =
          this.canvas.getWidth() - this.parametros.width * zoom;
      }
      if (vpt[5] >= 0) {
        this.canvas.viewportTransform[5] = 0;
      } else if (vpt[5] < this.canvas.getHeight() - this.parametros.height * zoom) {
        this.canvas.viewportTransform[5] =
          this.canvas.getHeight() - this.parametros.height * zoom;
      }
    },

    //PRONTO
    moveObject(event) {
      let a = parseInt(this.message);
      let objs = this.objetos.filter(function (obj) {
        return obj.id == a;
      });
      this.destaque = objs[0];
      var target = event.target;
      let targetID = ""+target.id
    

      if (this.destaque.type == "polygon" && !targetID.startsWith('i')) {
        this.destaque.points[target.id] = {
          x: target.getCenterPoint().x,
          y: target.getCenterPoint().y,
        };
        this.criaPontoInter();
      } else if (this.destaque.type == "rect"){
        let cordIni = { x: this.destaque.left, y: this.destaque.top };
        let cordFin = { x: this.destaque.left + this.destaque.width, y: this.destaque.top + this.destaque.height };

        if (targetID == "i") {
          cordIni.x = target.getCenterPoint().x;
          cordIni.y = target.getCenterPoint().y;
        }
        if (targetID == "f") {
          cordFin.x = target.getCenterPoint().x;
          cordFin.y = target.getCenterPoint().y;
        }

        //inicial
        this.destaque.set({ left: cordIni.x });
        this.destaque.set({ top: cordIni.y });

        //final
        this.destaque.set({ width: (cordIni.x - cordFin.x) * -1 });
        this.destaque.set({ height: (cordIni.y - cordFin.y) * -1 });
      }
    },

    //PRONTO
    mouseMove(event) {
      if(this.panning.enabled && event && event.e &&  this.panning.move){
        let delta =new fabric.Point(event.e.movementX, event.e.movementY);
        this.canvas.relativePan(delta);
        this.corrigeFoco(this.canvas.getZoom());
      }


      if (
        this.desenhando.linha &&
        this.desenhando.linha.class == "line" &&
        this.parametros.modo == 1
      ) {
        this.desenhando.linha.set({
          x2: event.absolutePointer.x,
          y2: event.absolutePointer.y,
        });
        this.canvas.renderAll();
        //desenha rastro linha
      } else if (this.parametros.modo == 2 && this.desenhando.poligono) {
        this.desenhando.poligono.set({
          width: (this.desenhando.poligono.left - event.absolutePointer.x) * -1,
        });
        this.desenhando.poligono.set({
          height: (this.desenhando.poligono.top - event.absolutePointer.y) * -1,
        });
        this.canvas.renderAll();
      }
    },

    //PRONTO
    mouseDown(event) {     
      let id = ""

      this.panning.lastX = event.e.clientX
      this.panning.lastY = event.e.lastY
      this.panning.move = true


      if(event.target){
        id = "" + event.target.id
      }
      

      if (event.target && this.desenhando.pontos.length > 0 && this.desenhando.pontos[0].id == event.target.id) {
        //fecha poligono
        this.terminaPoligono();
      }
      else if (this.parametros.modo == 1) {
        //comeca poligono, e adiciona os pontos
        this.adicionaPontoPoligono(event.absolutePointer);
      }
      else if (this.parametros.modo == 2) {
        //comeca o quadrado
        this.comecaQuadrado(event);
      } 
      else if (this.parametros.modo == 3 && event.target && id.startsWith('i')) {
        let indexf =  event.target.id.substring(1)
        let atual = this.destaque.points[indexf]
        let prox = this.destaque.points[indexf == this.destaque.points.length-1 ? 0 : parseInt(indexf)+1]

        let medio = { x: (parseInt(atual.x) + parseInt(prox.x))/2, y: (parseInt(atual.y) + parseInt(prox.y))/2 }


        let pontosnovos =  []
        this.destaque.points.forEach((element, index) => {
            pontosnovos.push(element)
            if (index == indexf){
              pontosnovos.push({x:medio.x,y:medio.y})
            }
        });

        this.destaque.points = pontosnovos

        this.canvas.renderAll()
        this.editaObjeto();
      }
    },

    //PRONTO
    mouseUp(event) {
      this.panning.move = false

      if (this.parametros.modo == 2) {
        this.terminaQuadrado(event);
      }
    },
    
    //PRONTO
    adicionaPontoPoligono(pontoAtual) {
      //adiciona um circulo na posicao clicada
      //gero um id unico pro ponto
      var random = Math.floor(Math.random() * (999999 - 99 + 1)) + 99;
      var id = new Date().getTime() + random;

      var circle = new fabric.Circle({
        radius: this.parametros.raioEsfera,
        fill: this.parametros.corEsfera,
        opacity: this.parametros.transparenciaEsfera,
        left: pontoAtual.x,
        top: pontoAtual.y,
        selectable: false,
        hasBorders: false,
        hasControls: false,
        originX: "center",
        originY: "center",
        id: id,
        objectCaching: false,
      });
      //verifico se e o primeiro ponto, e se for mudo a cor
      if (this.desenhando.pontos.length == 0) {
        circle.set({
          fill: this.parametros.corEsferaInicio,
        });
      }
      this.desenhando.pontos.push(circle);
      this.canvas.add(circle);

      //cria a linha de rastro
      var points = [pontoAtual.x, pontoAtual.y, pontoAtual.x, pontoAtual.y];
      let line = new fabric.Line(points, {
        strokeWidth: this.parametros.grossuraLinha,
        fill: this.parametros.corLinha,
        stroke: this.parametros.corLinha,
        opacity: this.parametros.transparenciaLinha,
        class: "line",
        originX: "center",
        originY: "center",
        selectable: false,
        hasBorders: false,
        hasControls: false,
        evented: false,
        objectCaching: false,
      });
      this.desenhando.linha = line;
      this.desenhando.linhas.push(line);
      this.canvas.add(line);
    },

    //PRONTO
    terminaPoligono() {
      var pontosTemp = [];

      //pego uma lista de coordenadas dos pontos, e apago os circulos
      this.desenhando.pontos.forEach((point, index) => {
        pontosTemp.push({
          x: point.left,
          y: point.top,
        });
        this.canvas.remove(point);
      });
      //apago as linhas desenhadas
      this.desenhando.linhas.forEach((linha, index) => {
        this.canvas.remove(linha);
      });

      //desenho o poligo final
      this.ultimaRef++;
      var polygon = new fabric.Polygon(pontosTemp, {
        fill: this.parametros.corPreenchimento,
        opacity: this.parametros.transparenciaPreenchimento,
        hasBorders: false,
        hasControls: false,
        id: this.ultimaRef,
        selectable: false,
        objectCaching: false,
      });
      this.canvas.add(polygon);
      this.objetos.push(polygon);
      this.mostraID();
      this.parametros.modo = 0;
    },

    //PRONTO
    comecaQuadrado(event) {
      let pontoAtual = event.absolutePointer;

      //adiciona um circulo na posicao clicada
      //gero um id unico pro ponto
      var random = Math.floor(Math.random() * (999999 - 99 + 1)) + 99;
      var id = new Date().getTime() + random;

      var circle = new fabric.Circle({
        radius: this.parametros.raioEsfera,
        fill: this.parametros.corEsferaInicio,
        opacity: this.parametros.transparenciaEsfera,
        left: pontoAtual.x,
        top: pontoAtual.y,
        selectable: false,
        hasBorders: false,
        hasControls: false,
        originX: "center",
        originY: "center",
        id: id,
        objectCaching: false,
      });
      this.desenhando.pontos.push(circle);
      this.canvas.add(circle);

      this.ultimaRef++;
      var rect = new fabric.Rect({
        left: pontoAtual.x,
        top: pontoAtual.y,
        fill: this.parametros.corPreenchimento,
        id: this.ultimaRef,
        opacity: this.parametros.transparenciaPreenchimento,
        selectable: false,
        objectCaching: false,
      });

      this.desenhando.poligono = rect;
      this.canvas.add(rect);
    },

    //PRONTO
    terminaQuadrado(event) {
      let pontoAtual = event.absolutePointer;
      this.desenhando.poligono.set({
        width: (this.desenhando.poligono.left - pontoAtual.x) * -1,
      });
      this.desenhando.poligono.set({
        height: (this.desenhando.poligono.top - pontoAtual.y) * -1,
      });
      this.canvas.renderAll();
      this.canvas.remove(this.desenhando.pontos[0]);
      this.objetos.push(this.desenhando.poligono);
      this.mostraID();
      this.parametros.modo = 0;
    },

    //PRONTO
    editaObjeto() {
      this.paraEdicao();
      this.parametros.modo = 3
      let a = parseInt(this.message);
      let objs = this.objetos.filter(function (obj) {
        return obj.id == a;
      });
      this.destaque = objs[0];
      if (this.destaque.type == "polygon") {
        this.destaque.points.forEach((element, index) => {
          var circle = new fabric.Circle({
            radius: this.parametros.raioEsfera,
            fill: this.parametros.corEsfera,
            opacity: this.parametros.transparenciaEsfera,
            left: element.x,
            top: element.y,
            hasBorders: false,
            hasControls: false,
            originX: "center",
            originY: "center",
            id: index,
            objectCaching: false,
          });
          if (this.edicaoCirculos.length == 0) {
            circle.set({
              fill: this.parametros.corEsferaInicio,
            });
          }

          this.canvas.add(circle);
          this.edicaoCirculos.push(circle);
        });
        this.criaPontoInter();
      } 
      
      else if (this.destaque.type == "rect") {
        var circleIni = new fabric.Circle({
          radius: this.parametros.raioEsfera,
          fill: this.parametros.corEsferaInicio,
          opacity: this.parametros.transparenciaEsfera,
          left: this.destaque.left,
          top: this.destaque.top,
          hasBorders: false,
          hasControls: false,
          originX: "center",
          originY: "center",
          id: "i",
          objectCaching: false,
        });
        this.canvas.add(circleIni);
        this.edicaoCirculos.push(circleIni);
        var circleFim = new fabric.Circle({
          radius: this.parametros.raioEsfera,
          fill:  this.parametros.corEsfera,
          opacity: this.parametros.transparenciaEsfera,
          left: this.destaque.left + this.destaque.width,
          top: this.destaque.top + this.destaque.height,
          hasBorders: false,
          hasControls: false,
          originX: "center",
          originY: "center",
          id: "f",
          objectCaching: false,
        });
        this.canvas.add(circleFim);
        this.edicaoCirculos.push(circleFim);
      }
    },

    //PRONTO
    tecladoEventUp(event) {
      //ctrl + z
      if (event.ctrlKey && event.key === "z") {
        this.desfaz();
      }
      if (event.keyCode === 27) {
        this.cancela();
      }
      if(event.keyCode === 16){
        console.log("desativa pan")
        this.panning.enabled = false
        this.canvas.defaultCursor = 'default';
      }
    },

    tecladoEventDown(event) {
      if(event.keyCode === 16){
        console.log("ativa pan")
        this.panning.enabled = true
        this.canvas.defaultCursor = 'move';
      }
    },

    //VERIFICAR
    inicia(image) {
      this.canvas.setBackgroundColor(
        {
          source: image,
          offsetX: 640,
          offsetY: 480,
        },
        this.canvas.renderAll.bind(this.canvas)
      );
    },

    //PRONTO
    adicionaPoligono() {
      this.desenhando = {
        objeto: null,
        linha: null,
        linhas: [],
        pontos: [],
      };
      this.parametros.modo = 1;
    },

    //PRONTO
    adicionaQuadrado() {
      this.desenhando = {
        objeto: null,
        linha: null,
        linhas: [],
        pontos: [],
      };
      this.parametros.modo = 2;
    },

    //PRONTO
    criaPontoInter(){
      if (this.destaque.type == "polygon") {
        this.pontosIntermendiarios.forEach((element, index) => {
          this.canvas.remove(element);
        });
        this.pontosIntermendiarios = [];
        this.destaque.points.forEach((element, index) => {
          let atual = this.destaque.points[index]
          let prox = this.destaque.points[index == this.destaque.points.length-1 ? 0 : parseInt(index)+1]
          let medio = { x: (parseInt(atual.x) + parseInt(prox.x))/2, y: (parseInt(atual.y) + parseInt(prox.y))/2 }
          var circle = new fabric.Circle({
            radius: this.parametros.raioEsferaInter,
            fill: this.parametros.corEsferaInter,
            opacity: this.parametros.transparenciaEsfera,
            left: medio.x,
            top: medio.y,
            hasBorders: false,
            hasControls: false,
            originX: "center",
            originY: "center",
            lockMovementX: false,
            lockMovementY:false,
            id: 'i'+index,
            objectCaching: false,
          });
          this.canvas.add(circle);
          this.pontosIntermendiarios.push(circle);
        });
      }
    },

    //PRONTO
    paraEdicao() {
      this.edicaoCirculos.forEach((point, index) => {
        this.canvas.remove(point);
      });
      this.edicaoCirculos = [];
      this.pontosIntermendiarios.forEach((point, index) => {
        this.canvas.remove(point);
      });
      this.pontosIntermendiarios = [];
      this.parametros.modo = 0;
    },

    //PRONTO
    apaga() {
      let forma = parseInt(this.message);

      let remover = this.objetos.filter(function (obj) {
        return parseInt(obj.id) == forma;
      });
      this.canvas.remove(remover[0]);
      this.objetos = this.objetos.filter(function (obj) {
        return parseInt(obj.id) != forma;
      });

      let remove = this.textos.filter(function (obj) {
        return parseInt(obj.id) == forma;
      });
      this.canvas.remove(remove[0]);
      this.textos = this.textos.filter(function (obj) {
        return parseInt(obj.id) != forma;
      });
    },

    //PRONTO
    showHide() {
      this.objetos.forEach((object, index) => {
        object.visible = !object.visible;
      });
      this.textos.forEach((object, index) => {
        object.visible = !object.visible;
      });
      this.canvas.renderAll();
    },

    //PRONTO
    destaca() {
      let a = parseInt(this.message);
      let objs = this.objetos.filter(function (obj) {
        return obj.id == a;
      });
      let objeto = objs[0];
      let textos = this.textos.filter(function (obj) {
        return obj.id == a;
      });
      let txt = textos[0];
      txt.fill = this.parametros.corDestaqueTexto;
      txt.stroke = this.parametros.corDestaqueTexto;
      txt.visible = true;
      objeto.fill = this.parametros.corDestaqueForma;
      objeto.stroke = this.parametros.corDestaqueForma;
      objeto.visible = true;
      this.canvas.renderAll();
    },

    //PRONTO
    cancela() {
      if (this.parametros.modo != 0) {
        this.desenhando.pontos.forEach((point, index) => {
          this.canvas.remove(point);
        });
        //apago as linhas desenhadas
        this.desenhando.linhas.forEach((linha, index) => {
          this.canvas.remove(linha);
        });
        this.canvas.remove(this.desenhando.poligono);
        if (this.parametros.modo == 3){
          this.paraEdicao();
        }
        this.modo = 0;
      }
    },

    //PRONTO
    desfaz() {
      if (this.parametros.modo == 1) {
        if (this.desenhando.pontos.length > 1) {
          //apago o ultimo ponto
          this.canvas.remove(
            this.desenhando.pontos[this.desenhando.pontos.length - 1]
          );
          this.desenhando.pontos.pop();
          this.canvas.remove(
            this.desenhando.pontos[this.desenhando.pontos.length - 1]
          );
          this.desenhando.pontos.pop();
          //apago a ultima linha
          this.canvas.remove(
            this.desenhando.linhas[this.desenhando.linhas.length - 1]
          );
          this.desenhando.linhas.pop();
          this.canvas.remove(
            this.desenhando.linhas[this.desenhando.linhas.length - 1]
          );
          let removido = this.desenhando.linhas.pop();
          this.adicionaPonto({ x: removido.x1, y: removido.y1 });
        } else {
          this.cancela();
          this.adicionaPoligono();
        }
      } else if (this.parametros.modo == 2) {
        this.cancela();
        this.adicionaQuadrado();
      }
    },

    //PRONTO
    mostraID() {
      if (this.parametros.mostraID) {
        this.objetos.forEach((obj, index) => {
          if (obj.type == "polygon") {
            var text = new fabric.Text("" + obj.id, {
              left: obj.points[0].x,
              top: obj.points[0].y,
              id: obj.id,
              fill: this.parametros.corTexto,
            });
            this.textos.push(text);
            this.canvas.add(text);
          } else if (obj.type == "rect") {
            var text1 = new fabric.Text("" + obj.id, {
              left: obj.left,
              top: obj.top,
              id: obj.id,
              fill: this.parametros.corTexto,
            });
            this.textos.push(text1);
            this.canvas.add(text1);
          }
        });
      }
    },

    //DEBUG
    //PRONTO
    listaDados() {
      this.canvas.setZoom(1);
      console.log("lista poligonos", this.objetos);
      console.log("ultimaRef", this.ultimaRef);
      console.log("zoom", this.canvas.getZoom());
      this.corrigeFoco(1);
    },
  },
};
</script>
