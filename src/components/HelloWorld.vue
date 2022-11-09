<template>
  <canvas ref="can" width="600" height="600"></canvas>
  <button @click="listaPoligonos">Lista Poligonos</button>
  <button @click="reiniciaPoligono">Reinicia Poligono</button>
  <button @click="EditaPoligono">Edita</button>
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
      if (zoom < 0.01) zoom = 0.01;
      this.canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
      opt.e.preventDefault();
      opt.e.stopPropagation();
    },
    moveObject(event){
      //
    },
    mouseMove(event){
      if(this.desenhando.linha && this.desenhando.linha.class == "line"){
        var pointer = this.canvas.getPointer(event.e);
        this.desenhando.linha.set({ x2: pointer.x, y2: pointer.y });
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

      //adiciona um circulo na posicao clicada
      //gero um id unico pro ponto
      var random = Math.floor(Math.random() * (999999 - 99 + 1)) + 99;
      var id = new Date().getTime() + random;
      
      var circle = new fabric.Circle({
        radius: 3,
        fill: '#ffffff',
        stroke: '#333333',
        strokeWidth: 0.5,
        left: (pontoAtual.x / this.canvas.getZoom()),
        top: (pontoAtual.y / this.canvas.getZoom()),
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
      var points = [(event.e.layerX / this.canvas.getZoom()), (event.e.layerY / this.canvas.getZoom()), (event.e.layerX / this.canvas.getZoom()), (event.e.layerY / this.canvas.getZoom())];
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
      var circulosTemp = [];
      console.log("edita",this.objetos[0].points)

      var random = Math.floor(Math.random() * (999999 - 99 + 1)) + 99;
      var id = new Date().getTime() + random;
      this.objetos[0].points.forEach((element, index) => {
        var circle = new fabric.Circle({
          radius: 3,
          fill: '#ffffff',
          stroke: '#333333',
          strokeWidth: 0.5,
          left: (element.x / this.canvas.getZoom()),
          top: (element.y / this.canvas.getZoom()),
          hasBorders: false,
          hasControls: false,
          originX: 'center',
          originY: 'center',
          id: id,
          objectCaching: false
        });
        this.canvas.add(circle);
        circulosTemp.push(circle);
      });
      var lastControl = this.objetos[0].points.length - 1;
      this.objetos[0].points.reduce(function (acc, point, index) {
				acc['p' + index] = new fabric.Control({
					positionHandler: this.polygonPositionHandler,
					actionHandler: this.anchorWrapper(index > 0 ? index - 1 : lastControl, this.actionHandler),
					actionName: 'modifyPolygon',
					pointIndex: index
				});
				return acc;
			}
			, {});
      console.log("polly controls",this.objetos[0].controls)
      this.canvas.requestRenderAll();
    },
    anchorWrapper(anchorIndex, fn) {
      console.log("anchorWrapper");
      return function (eventData, transform, x, y) {
        var fabricObject = transform.target,
        absolutePoint = fabric.util.transformPoint({
          x: (fabricObject.points[anchorIndex].x - fabricObject.pathOffset.x),
          y: (fabricObject.points[anchorIndex].y - fabricObject.pathOffset.y),
        }, fabricObject.calcTransformMatrix()),
        actionPerformed = fn(eventData, transform, x, y),
        newDim = fabricObject._setPositionDimensions({}),
        polygonBaseSize = this.getObjectSizeWithStroke(fabricObject),
        newX = (fabricObject.points[anchorIndex].x - fabricObject.pathOffset.x) / polygonBaseSize.x,
        newY = (fabricObject.points[anchorIndex].y - fabricObject.pathOffset.y) / polygonBaseSize.y;
        fabricObject.setPositionByOrigin(absolutePoint, newX + 0.5, newY + 0.5);
        return actionPerformed;
      }
    },
    
    polygonPositionHandler(dim, finalMatrix, fabricObject) {
      console.log("polygonPositionHandler");
      var x = (fabricObject.points[this.pointIndex].x - fabricObject.pathOffset.x),
      y = (fabricObject.points[this.pointIndex].y - fabricObject.pathOffset.y);
      return fabric.util.transformPoint({ x: x, y: y },
      fabric.util.multiplyTransformMatrices(
        fabricObject.canvas.viewportTransform,
        fabricObject.calcTransformMatrix()
      ));
    }, 

    actionHandler(eventData, transform, x, y) {
      console.log("actionHandler");
      var polygon = transform.target,
      currentControl = polygon.controls[polygon.__corner],
      mouseLocalPosition = polygon.toLocalPoint(new fabric.Point(x, y), 'center', 'center'),
      polygonBaseSize = this.getObjectSizeWithStroke(polygon),
      size = polygon._getTransformedDimensions(0, 0),
      finalPointPosition = {
        x: mouseLocalPosition.x * polygonBaseSize.x / size.x + polygon.pathOffset.x,
        y: mouseLocalPosition.y * polygonBaseSize.y / size.y + polygon.pathOffset.y
      };
      polygon.points[currentControl.pointIndex] = finalPointPosition;
      return true;
    },
    getObjectSizeWithStroke(object) {
      var stroke = new fabric.Point(
        object.strokeUniform ? 1 / object.scaleX : 1,
        object.strokeUniform ? 1 / object.scaleY : 1
      ).multiply(object.strokeWidth);
      return new fabric.Point(object.width + stroke.x, object.height + stroke.y);
    },

  },
};


</script>



