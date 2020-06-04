
<style scoped>
* {
  font-size: 0;
  touch-action: pan-y; 
}
.box {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  text-align: center;
}
</style>

<template>
  <div class="box">
    <canvas ref="bargraphCanvas" :width="canvasWidth" :height="canvasHeight"></canvas>
  </div>
</template>

<script>
import conf from '../../static/conf'
import axios from 'axios'
export default {
  name: 'Scan',
  data () { 
    return {
      canvasWidth: document.body.clientWidth, // 画布大小
      canvasHeight: document.body.clientHeight,
      myCanvas: null,
      ctx: null,

      imgX: 0, // 图片在画布中渲染的起点x坐标
      imgY: 60,
      MINIMUM_SCALE: 0.2,
      imgScale: 0.2, // 图片启示的缩放大小
      extraImg: {url:require("../../static/images/covers/map.png")},
      loadImgObj: null,

      img: null,
      flag: false,
      pos: {},
      posl: {},
      
      flagPC: true,
    }
  },
  watch: {
    
  },
  mounted() {
    this.canvasHeight = document.body.clientHeight;
    this.flagPC = this.IsPC()
    this.myCanvas = this.$refs.bargraphCanvas;
    this.ctx = this.myCanvas.getContext('2d');
    this.loadImg();
    this.canvasEventsInit();
  },
  destroyed() {
      
  },
  methods: {
    loadImg() {
      let _this = this;
      _this.img = new Image();
      _this.img.src = this.extraImg.url
      _this.img.onload = function () {
          _this.drawImage();
      }
      // _this.imgX = _this.img.width*_this.imgScale/4
    },

    drawImage() {
      let _this = this;
      this.ctx.clearRect(0, 0, this.myCanvas.width, this.myCanvas.height);
      // // 保证  imgX  在  [img.width*(1-imgScale),0]   区间内
      // if(_this.imgX<_this.img.width*(1-_this.imgScale)) {
      //     _this.imgX = _this.img.width*(1-_this.imgScale);
      // }
      // else if(_this.imgX>0) {
      //     _this.imgX=0
      // }
      // // 保证  imgY   在  [img.height*(1-imgScale),0]   区间内
      // if(_this.imgY<_this.img.height*(1-_this.imgScale)) {
      //     _this.imgY = _this.img.height*(1-_this.imgScale);
      // }else if(_this.imgY>0) {
      //     _this.imgY=0
      // }

      this.ctx.drawImage(
          _this.img, //规定要使用的图像、画布或视频。
          0, 0, //开始剪切的 x 坐标位置。
          _this.img.width, _this.img.height,  //被剪切图像的高度。
          _this.imgX, _this.imgY,//在画布上放置图像的 x 、y坐标位置。
          _this.img.width * _this.imgScale, _this.img.height * _this.imgScale  //要使用的图像的宽度、高度
      );
    },

    canvasEventsInit() {
      var pageX, pageY, initX, initY;
      var start = [];
      let _this = this;
      console.log(this.flagPC)
      if(this.flagPC) {
        //PC
        this.myCanvas.onmousedown = function (event) {
          _this.flag = true;
          _this.pos = _this.windowToCanvas(event.clientX, event.clientY);  //坐标转换，将窗口坐标转换成canvas的坐标

        };
        this.myCanvas.onmousemove = function (evt) {  //移动
            if(_this.flag ){
                // console.log(evt)
                _this.posl = _this.windowToCanvas(evt.clientX, evt.clientY);
                var x = _this.posl.x - _this.pos.x, y = _this.posl.y - _this.pos.y;
                _this.imgX  += x;
                _this.imgY  += y;
                _this.pos = JSON.parse(JSON.stringify(_this.posl));
                _this.drawImage();  //重新绘制图片
            }

        };
        this.myCanvas.onmouseup = function () {
            _this.flag  = false;
        };
        this.myCanvas.onmousewheel = this.myCanvas.onwheel = function (event) {    //滚轮放大缩小
            var pos = _this.windowToCanvas (event.clientX, event.clientY);
            // event.wheelDelta = event.wheelDelta ? event.wheelDelta : (event.deltalY * (-40));  //获取当前鼠标的滚动情况
            var newPos = {x:((pos.x-_this.imgX)/_this.imgScale).toFixed(2) , y:((pos.y-_this.imgY)/_this.imgScale).toFixed(2)};
            if (event.wheelDelta > 0) {// 放大
                    _this.imgScale +=0.05;
                    _this.imgX = (1-_this.imgScale)*newPos.x+(pos.x-newPos.x);
                    _this.imgY = (1-_this.imgScale)*newPos.y+(pos.y-newPos.y);
            } else {//  缩小
                _this.imgScale -=0.05;
                if(_this.imgScale<_this.MINIMUM_SCALE) {//最小缩放1
                    _this.imgScale = _this.MINIMUM_SCALE;
                }
                _this.imgX = (1-_this.imgScale)*newPos.x+(pos.x-newPos.x);
                _this.imgY = (1-_this.imgScale)*newPos.y+(pos.y-newPos.y);
                // console.log(_this.imgX,_this.imgY);
            }
            _this.drawImage();   //重新绘制图片
        };
      } else {
        //Phone
        this.myCanvas.ontouchstart = function (event) {
          _this.flag = true;
          if(event.touches && event.touches.length < 2) {
            let touch = event.touches[0];
            _this.pos = _this.windowToCanvas(touch.clientX, touch.clientY);  //坐标转换，将窗口坐标转换成canvas的坐标
          }else{
            let touches = event.touches;
            //手指按下时的手指所在的X，Y坐标  
            pageX = touches[0].pageX;
            pageY = touches[0].pageY;
            //初始位置的X，Y 坐标  
            initX = event.target.offsetLeft;
            initY = event.target.offsetTop;
            //记录初始 一组数据 作为缩放使用
            if (touches.length >= 2) { //判断是否有两个点在屏幕上
              start = touches; //得到第一组两个点
            };
          }
        };
        this.myCanvas.ontouchmove = function (evt) {  //移动
            if(_this.flag ){
              if(evt.touches && evt.touches.length < 2) {
                // console.log(evt)
                let touch = evt.touches[0];
                _this.posl = _this.windowToCanvas(touch.clientX, touch.clientY);
                var x = _this.posl.x - _this.pos.x, y = _this.posl.y - _this.pos.y;
                _this.imgX  += x;
                _this.imgY  += y;
                _this.pos = JSON.parse(JSON.stringify(_this.posl));
              }else{
                let touches = evt.touches;
                // 2 根 手指执行 目标元素放大操作
                //得到第二组两个点
                var now = touches;
                var pos = _this.windowToCanvas (now[0].clientX, now[0].clientY);
                var newPos = {x:((pos.x-_this.imgX)/_this.imgScale).toFixed(2) , y:((pos.y-_this.imgY)/_this.imgScale).toFixed(2)};
                // Math.abs(touches[0].pageX-touches[1].pageX)
                //当前距离变小， getDistance 是勾股定理的一个方法
                if(_this.getDistance(now[0], now[1]) < _this.getDistance(start[0], start[1])){
                  // 缩小
                  _this.imgScale -=0.03;
                  if(_this.imgScale<_this.MINIMUM_SCALE) {//最小缩放1
                      _this.imgScale = _this.MINIMUM_SCALE;
                  }
                  _this.imgX = (1-_this.imgScale)*newPos.x+(pos.x-newPos.x);
                  _this.imgY = (1-_this.imgScale)*newPos.y+(pos.y-newPos.y);
                  // console.log(_this.imgX,_this.imgY);
                }else if(_this.getDistance(now[0], now[1]) > _this.getDistance(start[0], start[1])){
                  // 放大
                  if(_this.imgScale < 1) {
                    _this.imgScale +=0.03;
                      _this.imgX = (1-_this.imgScale)*newPos.x+(pos.x-newPos.x);
                      _this.imgY = (1-_this.imgScale)*newPos.y+(pos.y-newPos.y);
                  }
                }
                start = now;
              }
              _this.drawImage();  //重新绘制图片
            }

        };
        this.myCanvas.ontouchend = function () {
            _this.flag  = false;
        };
      }
    },

    /*坐标转换*/
    windowToCanvas(x,y) {
        var box = this.myCanvas.getBoundingClientRect();  //这个方法返回一个矩形对象，包含四个属性：left、top、right和bottom。分别表示元素各边与页面上边和左边的距离
        return {
            x: x - box.left - (box.width - this.myCanvas.width) / 2,
            y: y - box.top - (box.height - this.myCanvas.height) / 2
        };
    },

    //缩放 勾股定理方法-求两点之间的距离
    getDistance(p1, p2) {
        var x = p2.pageX - p1.pageX,
        y = p2.pageY - p1.pageY;
        return Math.sqrt((x * x) + (y * y));
    },

    IsPC() {
      var userAgentInfo = navigator.userAgent;
      var Agents = ["Android", "iPhone",
                  "SymbianOS", "Windows Phone",
                  "iPad", "iPod"];
      var flag = true;
      for (var v = 0; v < Agents.length; v++) {
          if (userAgentInfo.indexOf(Agents[v]) > 0) {
              flag = false;
              break;
          }
      }
      return flag;
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
