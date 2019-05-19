<template>
  <div>
    <el-button type="primary" @click="visible=true" size="mini">{{label}}</el-button>
    <el-dialog
      title="裁剪图片"
      :visible.sync="visible"
      :width="boxWidth+40+'px'"
      :before-close="handleClose">
      <div class="toolMain">
        <div ref="toolBox"
             :style="'height:'+boxHeight+'px;width:'+boxWidth+'px'"
             v-on:mousemove="controlBtnMouseMove"
             v-on:mouseup="controlBtnMouseUp"
             v-on:mouseleave="controlBtnMouseLeave"
             class="toolBox">
          <div class="tips" v-show="!drawImg.img">
            <el-button @click="chooseImg" type="warning" size="mini">请选择要裁剪的图片</el-button>
          </div>
          <div
            v-show="drawImg.img"
            v-on:mousedown="toolBoxMouseDown"
            v-on:mouseup="toolBoxMouseUp"
            v-on:mousemove="toolBoxMouseMove"
            v-on:mouseleave="toolBoxMouseLeave"
            ref="toolBoxControl"
            class="toolBoxControl">
            <div class="controlBox">
              <div class="selectArea">宽:{{toolBox.width}} 高:{{toolBox.height}} (x:{{toolBox.boxMove.moveTo.x}},y:{{toolBox.boxMove.moveTo.y}})</div>
              <div data-name="leftUp"
                   v-on:mousedown="controlBtnMouseDown"
                   class="leftUp controlBtn"></div>
              <div ref="controlBtn"
                   data-name="rightDown"
                   v-on:mousedown="controlBtnMouseDown"
                   class="rightDown controlBtn"></div>
            </div>
          </div>
          <canvas class="canvasSelectBox" ref="canvasSelectBox" :width="boxWidth" :height="boxHeight"></canvas>
          <canvas class="canvas" ref="canvas" :width="boxWidth" :height="boxHeight"></canvas>
        </div>
        <div class="toolBar">
          <el-form size="mini" :inline="true">
            <el-form-item v-if="rate" label="图片宽高比：">
              <el-input width="70" readonly v-model="rate" placeholder="width/height"></el-input>
            </el-form-item>
              <el-form-item>
                  <input v-show="false" @change="putImgToCanv" ref="inputFile" type="file" accept="image/*">
                  <el-button @click="chooseImg" type="primary" plain>选择图片</el-button>
              </el-form-item>
          </el-form>
        </div>
      </div>
      <span slot="footer" class="dialog-footer">
    <el-button @click="handleClose">取消</el-button>
    <el-button :disabled="!drawImg.img" type="primary" @click="cropPicture">确定</el-button>
  </span>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    name:'CropTool',
      props:{
          label:{
              type:String,
              default:"选择图片",
              required:false
          },
          boxWidth:{
              type:Number,
              default:800,
              required:false
          },
          boxHeight:{
              type:Number,
              default:400,
              required:false
          },
          rate:{
              type:String,
              default:null,
              required:false,
          }
      },
    data(){
      return {
        visible:false,
        drawImg:{
          img:null,//规定要使用的图像、画布或视频
          sx:0,//开始剪切的 x 坐标位置
          sy:0,//开始剪切的 y 坐标位置
          swidth:0,//被剪切图像的宽度
          sheight:0,//被剪切图像的高度
          x:0,//在画布上放置图像的 x 坐标位置
          y:0,//在画布上放置图像的 y 坐标位置
          width:0,//要使用的图像的宽度
          height:0//要使用的图像的高度
        },
        toolBox:{
          disable:true,
          width:200,
          height:200,
          x:0,
          y:0,
          boxMove:{
            start:{
              x:0,
              y:0,
            },
            moveTo:{
              x:0,
              y:0,
            }
          },
        },
        controlBox:{
          disable:true,
          btnName:'',
          start:{
            x:0,
            y:0,
            width:0,
            height:0,
          }
        },
        selectBox:false,
        selectBoxColor:'rgba(0,0,0,0.6)',
      }
    },
    mounted(){

    },
    methods:{
      handleClose:function(){
        this.visible = false;
        this.$nextTick(()=>{
          this.clearAll();
        });
      },

      // 选择图片 e.stopPropagation();
      chooseImg:function(){
        this.$refs['inputFile'].value = "";
        this.drawImg.img = null;
        this.$refs['inputFile'].click();
      },
      // 将选择的图片绘制到画布
      putImgToCanv:function(e){
        let _this = this;
        let file = e.target.files[0] || null;
        if(file) {
          let reader = new FileReader();
          new Image();
          reader.readAsDataURL(file);
          reader.onload = function(e) {
            // 图片base64化
            let newUrl = e.target.result;
            let img = document.createElement('img');
            img.src = newUrl;
            let timmer = setInterval(function(){
              if(reader.readyState == 2) {
                clearInterval(timmer);
                let imgHeight = img.height;
                let imgWidth = img.width;
                let boxWidth = _this.boxWidth;
                let boxHeight = _this.boxHeight;
                let c = _this.$refs['canvas'];
                let ctx = c.getContext("2d");
                let rate;
                let drawImg = _this.drawImg;
                drawImg.img = img;
                if(imgHeight<boxHeight && imgWidth < boxWidth) {
                  rate = 1;
                  drawImg.x = (boxWidth - imgWidth)/2;
                  drawImg.y = (boxHeight - imgHeight)/2;
                } else {
                  if(imgWidth/imgHeight <= boxWidth/boxHeight) { // 计算宽高比
                    rate = boxHeight/imgHeight;
                    drawImg.x = (boxWidth - imgWidth*rate)/2;
                  } else {
                    rate = boxWidth/imgWidth;
                    drawImg.y = (boxHeight - imgHeight*rate)/2;
                  }
                }
                ctx.clearRect(0,0,c.width,c.height);
                drawImg.swidth = imgWidth;
                drawImg.sheight = imgHeight;
                drawImg.width = imgWidth*rate;
                drawImg.height = imgHeight*rate;
                ctx.drawImage(drawImg.img,drawImg.sx,drawImg.sy,drawImg.swidth,drawImg.sheight,drawImg.x,drawImg.y,drawImg.width,drawImg.height);
              }
            },200);
          };
            this.toolBox.x = this.boxWidth/2 - this.toolBox.width;
            this.toolBox.y = this.boxHeight/2 - this.toolBox.height;
            this.drawControlBox(this.toolBox.width,this.toolBox.height,this.toolBox.x,this.toolBox.y);
        }
      },
      // clear both
      clearAll:function(){
        let _this = this;
        let c = _this.$refs['canvas'];
        let ctx = c.getContext("2d");
        ctx.clearRect(0,0,c.width,c.height);
        let c1 = _this.$refs['canvasSelectBox'];
        let ctx1 = c1.getContext("2d");
        ctx1.clearRect(0,0,c1.width,c1.height);
        this.drawImg = {
          img:null,//规定要使用的图像、画布或视频
            sx:0,//开始剪切的 x 坐标位置
            sy:0,//开始剪切的 y 坐标位置
            swidth:0,//被剪切图像的宽度
            sheight:0,//被剪切图像的高度
            x:0,//在画布上放置图像的 x 坐标位置
            y:0,//在画布上放置图像的 y 坐标位置
            width:0,//要使用的图像的宽度
            height:0//要使用的图像的高度
        };
      },
      // draw control
      drawControlBox:function(width,height,x,y){
        if(width<50|| height<50) {
          return false;
        }
        let $toolBoxControl = this.$refs['toolBoxControl'];
        if(x<0) {
          x = 0;
        }
        if(y<0) {
          y=0;
        }
        if(x+this.toolBox.width > this.boxWidth) {
          x = this.boxWidth - this.toolBox.width;
        }
        if(y+this.toolBox.height > this.boxHeight) {
          y = this.boxHeight - this.toolBox.height;
        }
        $toolBoxControl.style.top = y + 'px';
        $toolBoxControl.style.left = x + 'px';

        let c=this.$refs['canvasSelectBox'];
        let ctx=c.getContext("2d");
        ctx.fillStyle = this.selectBoxColor;
        ctx.clearRect(0,0,c.width,c.height);
        ctx.fillRect(0,0,c.width,c.height);

        if(this.rate) {
          let p = this.rate.split(':')[0] / this.rate.split(':')[1];
          ctx.clearRect(x,y,width,width/p);
          $toolBoxControl.style.width = width+'px';
          $toolBoxControl.style.height = width/p+'px';
          this.toolBox.height = width;
        } else {
          ctx.clearRect(x,y,width,height);
          $toolBoxControl.style.width = width+'px';
          $toolBoxControl.style.height = height+'px';
        }
        this.toolBox.boxMove.moveTo.x = x;
        this.toolBox.boxMove.moveTo.y = y;
      },
      // toolBoxMouseDown
      toolBoxMouseDown:function(e){
        let $toolBox = this.$refs['toolBoxControl'];
        this.toolBox.x = parseInt($toolBox.style.left.split('px')[0]);
        this.toolBox.y = parseInt($toolBox.style.top.split('px')[0]);
        this.toolBox.disable = false;
        this.toolBox.boxMove.start = {
          x:e.pageX,
          y:e.pageY,
        };
      },
      toolBoxMouseMove:function(e){
        if(this.toolBox.disable === false) {
          let offsetX = e.pageX - this.toolBox.boxMove.start.x;
          let offsetY = e.pageY - this.toolBox.boxMove.start.y;

          let x = this.toolBox.x + offsetX;
          let y = this.toolBox.y + offsetY;
          this.drawControlBox(this.toolBox.width,this.toolBox.height,x,y);
        }
      },
      toolBoxMouseLeave:function(){
        this.toolBox.disable = true;
      },
      toolBoxMouseUp:function(e){
        let $toolBox = this.$refs['toolBoxControl'];
        this.toolBox.x = parseInt($toolBox.style.left.split('px')[0]);
        this.toolBox.y = parseInt($toolBox.style.top.split('px')[0]);
        this.toolBox.disable = true;
      },
      // control box
      controlBtnMouseDown:function(e){
        this.controlBox.disable = false;
        this.controlBox.btnName = e.target.dataset.name;
        this.controlBox.start.x = e.clientX;
        this.controlBox.start.y = e.clientY;
        this.controlBox.start.width = this.toolBox.width;
        this.controlBox.start.height = this.toolBox.height;
        e.stopPropagation();
      },
      controlBtnMouseUp:function(e){
        this.controlBox.disable = true;
        e.stopPropagation();
      },
      controlBtnMouseLeave:function(e){
        this.controlBox.disable = true;
        e.stopPropagation();
      },
      controlBtnMouseMove:function(e){
        if(this.controlBox.disable === false) {
          let offsetX = e.clientX - this.controlBox.start.x;
          let offsetY = e.clientY - this.controlBox.start.y;

          if(this.controlBox.btnName=='leftUp') {
            let x = this.toolBox.x + offsetX;
            let y = this.toolBox.y  + offsetY;
            this.toolBox.width = (this.controlBox.start.width - offsetX)>50?(this.controlBox.start.width - offsetX):50;
            this.toolBox.height = (this.controlBox.start.height - offsetY)>50?(this.controlBox.start.height - offsetY):50;
            this.drawControlBox(this.toolBox.width,this.toolBox.height,x,y);
          }

          if(this.controlBox.btnName=='rightDown') {
            let x = this.toolBox.x;
            let y = this.toolBox.y;
            this.toolBox.width = (this.controlBox.start.width + offsetX)>50?(this.controlBox.start.width + offsetX):50;
            this.toolBox.height = (this.controlBox.start.height + offsetY)>50?(this.controlBox.start.height + offsetY):50;
            this.drawControlBox(this.toolBox.width,this.toolBox.height,x,y);
          }
        }
        e.stopPropagation();
      },
      cropPicture:function(){
        let _this = this;
        // get img
        let c = this.$refs['canvas'];
        let tempImg = new Image();
        tempImg.src = c.toDataURL();
        c.toBlob((blob)=>{
          let reader = new FileReader();
          reader.readAsDataURL(blob);
          reader.onload = function(){
            let timmer = setInterval(function(){
              if(reader.readyState == 2) {
                clearInterval(timmer);
                let newCanv = document.createElement('canvas');
                newCanv.width = _this.toolBox.width;
                newCanv.height = _this.toolBox.height;
                let ctx = newCanv.getContext("2d");
                let params = _this.toolBox;
                if(this.rate) {
                  let p = this.rate.split(':')[0] / this.rate.split(':')[1];
                  ctx.drawImage(tempImg,params.x,params.y,params.width,params.width*p,0,0,params.width,params.width*p);
                } else {
                  ctx.drawImage(tempImg,params.x,params.y,params.width,params.height,0,0,params.width,params.height);
                }
                _this.handleClose();
                  newCanv.toBlob(function(blob){
                      _this.$emit('cutDown',{
                          blob:blob,
                          src:newCanv.toDataURL()
                      })
                  });
              }
            },200);
          };
        });
      },
    }
  }
</script>
<style scoped>
  .toolMain {
    box-sizing: border-box;
  }
  .toolBox {
    border:1px solid #dedede;
    background-image: linear-gradient(45deg,rgba(0,0,0,.25) 25%,transparent 0,transparent 75%,rgba(0,0,0,.25) 0),
    linear-gradient(45deg,rgba(0,0,0,.25) 25%,transparent 0,transparent 75%,rgba(0,0,0,.25) 0);
    background-color: #eee;
    background-size: 30px 30px;
    background-position: 0 0,15px 15px;
    position: relative;
  }
  .canvas {
    position: absolute;
    top:0;
    left:0;
    z-index:99;
  }
  .canvasSelectBox {
    position: absolute;
    top:0;
    left:0;
    z-index:100;
  }
  .toolBoxControl {
    background:transparent;
    position:absolute;
    z-index:101;
    box-sizing:border-box;
    border:1px dashed #409EFF;
  }
  .controlBox {
    width:100%;
    height:100%;
    position: absolute;
    cursor:move;
  }
  .controlBtn {
    width:10px;
    height:10px;
    box-sizing:border-box;
    border:1px solid #409EFF;
    background:#409EFF;
    position: absolute;
  }
  .leftUp {
    top:0;
    left:0;
    margin-left:-5px;
    margin-top:-5px;
    cursor:se-resize;
  }
  .leftDown {
    bottom:0;
    left:0;
    margin-left:-5px;
    margin-bottom:-5px;
    cursor:sw-resize;
  }
  .rightUp {
    top:0;
    right:0;
    margin-right:-5px;
    margin-top:-5px;
    cursor:sw-resize;
  }
  .rightDown {
    bottom:0;
    right:0;
    margin-right:-5px;
    margin-bottom:-5px;
    cursor:se-resize;
  }
  .toolBar {
    margin-top:20px;
  }
  .selectArea {
    display:block;
    width:260px;
    text-align:right;
    color:#fff;
    position:absolute;
    top:-20px;
    right:0;
    font-size:10px;
    user-select: none;
  }
  .tips {
    position:absolute;
    top:50%;
    left:50%;
    color:red;
    z-index:101;
    transform: translate(-50%,-50%);
  }
</style>
