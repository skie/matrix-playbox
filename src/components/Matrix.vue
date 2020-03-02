<template>
  <div class="wrap">
    <div class="inputs" style="float:right; margin-right: 30%;">
      <div>
        <label for="effect" >Effect:&nbsp;</label>
        <select id="effect" v-model="effectId" @change="afterChange">
          <option v-for="option in types" :key="option.value" :value="option.value">
            {{ option.text }}
          </option>
        </select>
      </div>
      <div>
        <label for="blur" >Blur:&nbsp;</label>
        <select id="blur"  v-model="blur" >
          <option v-for="(val,id) in blurs" :key="id" :value="val">
            {{ val }}
          </option>
        </select>
      </div>
      <div>
        <label for="brightness">Brightness:&nbsp;</label>
        <select id="brightness" v-model="brightness" >
          <option v-for="(val,id) in brightnesses" :key="id" :value="val">
            {{ val }}
          </option>
        </select>
      </div>
      <div>
        <label for="contrast" >Contrast:&nbsp;</label>
        <select id="contrast" v-model="contrast" >
          <option v-for="(val,id) in contrasts" :key="id" :value="val">
            {{ val }}
          </option>
        </select>
      </div>
    </div>
    <div class="matrix-wrap" :style="wrapStyle()">
      <div class="matrix" v-if="renderComponent">
        <div class="point" v-for="(item,idx) in area" :key="idx" :style="color(item)" >
        </div>
      </div>
    </div>
  </div>
</template>

<script>
var colorsys = require('colorsys')

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  methods: {
    wrapStyle() {
        return {
            filter: 'blur('+this.blur+'px) contrast('+this.contrast+'%) brightness('+this.brightness+')'
        };
    },
    drawPixel(x,y,color) {
        if (x>= this.width) x=x%this.width;
        if (y>= this.height) y=y%this.height;
        this.area[x*this.width+y] = colorsys.stringify(color);
    },
    drawPixelRaw(x,y,color) {
        if (x>= this.width) x=x%this.width;
        if (y>= this.height) y=y%this.height;
        this.area[x*this.width+y] = color;
    },
    getPixelColor(x,y) {
        if (x>= this.width) x=x%this.width;
        if (y>= this.height) y=y%this.height;
        return this.area[x*this.width+y]
    },
    drawPixelI(y,x,color) {
        this.area[x*this.width+y] = colorsys.stringify(color);
    },
    drawPixelRawI(y,x,color) {
        this.area[x*this.width+y] = color;
    },
    getPixelColorI(y,x) {
        return this.area[x*this.width+y]
    },
    drawPixelI2(y,x,color) {
        // y = this.height - y
        x = this.height - x
        this.area[x*this.width+y] = colorsys.stringify(color);
    },
    drawPixelRawI2(y,x,color) {
        // y = this.height - y
        x = this.height - x
        this.area[x*this.width+y] = color;
    },
    getPixelColorI2(y,x) {
        // y = this.height - y
        x = this.height - x
        return this.area[x*this.width+y]
    },

    rainbowVertical() {
      this.renderComponent = false;
      this.hue += 2;
      for (let j = 0; j < this.height; j++) {
        let thisColor = { h: ((this.hue + j * this.scale)%360) , s: 70 , l: 50 };
        for (let i = 0; i < this.width; i++)
          this.drawPixel(i, j, thisColor);
      }
      this.forceRerender()
    },

    afterChange() {
      let black = {h: 0,s: 0,l: 0};
      this.drawRect(0,0,15,15,black);
    },

    drawSquare(x0, y0, r, color) {
      for (let x = x0 - r; x <= x0 + r; x++) {
        for (let y = y0 - r; y <= y0 + r; y++) {
            this.drawPixel(x, y, color);
        }
      }
    },

    drawPoint(x0, y0, r, color) {
      for (let x = x0 - r; x <= x0 + r; x++) {
        let d1 = Math.abs(x - x0)
        for (let y = y0 - r; y <= y0 + r; y++) {
            let d2 = Math.abs(y - y0)
            let s1 = r * r
            let sq = (s1 - d1 * d2)
            let v1 = sq / s1;
            let c1 = {
                h: color.h,
                s: color.s,
                l: Math.round(color.l * v1)};
            this.drawPixel(x, y, c1);
        }
      }
    },

    randMovePoint(i) {
      let p = this.points[i]
      let black = {h: 0,s: 0,l: 0};
      this.drawSquare(p.x, p.y, p.r, black);

      p.x += this.getRandomInt(2) - 1
      if (p.x < 0) p.x += this.width
      if (p.x >= this.width) p.x -= this.width
      p.y += this.getRandomInt(2) - 1
      if (p.y < 0) p.y += this.height
      if (p.y >= this.height) p.y -= this.height
      p.hue += 20;
      p.hue = p.hue % 360;
      this.points[i] = p;
      let c0 = {h: p.hue, s: 70, l: 70};
      this.drawPoint(p.x, p.y, p.r, c0);
    },

    horMovePoint(i, dir) {
      let p = this.points[i]
      let black = {h: 0,s: 0,l: 0};
      this.drawSquare(p.x, p.y, p.r, black);

      if (dir == 1) {
        p.x += this.getRandomInt(2)
      } else {
          p.x -= this.getRandomInt(2)
      }
      if (p.x < 0) p.x += this.width
      if (p.x >= this.width) p.x -= this.width
      // p.y += this.getRandomInt(4) - 2
      // if (p.y < 0) p.y += this.height
      // if (p.y >= this.height) p.y -= this.height
      p.hue += 20;
      p.hue = p.hue % 360;
      this.points[i] = p;
      let c0 = {h: p.hue, s: 70, l: 70};
      this.drawPoint(p.x, p.y, p.r, c0);
    },

    horMoveSquare(i, dir) {
      let p = this.points[i]
      let black = {h: 0,s: 0,l: 0};
      this.drawSquare(p.x, p.y, p.r, black);

      if (dir == 1) {
        p.x += 1
      } else {
          p.x -= 1
      }
      if (p.x < 0) p.x += this.width
      if (p.x >= this.width) p.x -= this.width
      // p.y += this.getRandomInt(4) - 2
      // if (p.y < 0) p.y += this.height
      // if (p.y >= this.height) p.y -= this.height
      p.hue += 5;
      p.hue = p.hue % 360;
      // console.log(p.hue)
      this.points[i] = p;
      let c0 = {h: p.hue, s: 70, l: 50};
      this.drawSquare(p.x, p.y, p.r, c0);
    },


    roundCircles() {
      this.renderComponent = false;
      this.hue += this.scale;
      this.randMovePoint(0);
      this.randMovePoint(1);
      this.randMovePoint(2);
      this.forceRerender()
    },

    horizCircles() {
      this.renderComponent = false;
      this.hue += this.scale;
      this.horMovePoint(0, 0);
      this.horMovePoint(2, 1);
      this.forceRerender()
    },

    drawRect(x0,y0,x1,y1,color) {
      for (let x = x0; x <= x1; x++) {
        for (let y = y0; y <= y1; y++) {
            this.drawPixel(x, y, color);
        }
      }
    },

    horizSquares() {
      this.renderComponent = false;
      this.hue += this.scale;
      this.horMovePoint(0, 0);
      this.horMovePoint(2, 1);
      this.hue += 2;
      this.l += this.getRandomInt(4) - 2;
      if (this.l>80) this.l = 50;
      if (this.l<30) this.l = 50;
      this.drawRect(0,7,15,8, {h: this.hue, s: 80, l: this.l})
      this.forceRerender()
    },


    runningLines() {
      this.renderComponent = false;
      for (let i = 0; i < this.width; i++) {
        this.lineHues[i] += 8;
      }
      for (let j = 0; j < this.height; j++) {
        this.linePoses[j] += this.getRandomInt(2);
      }
      for (let i = 0; i < this.width; i++) {
        for (let j = 0; j < this.height; j++) {
          let l0 = (j + this.linePoses[i]) % 16;
          let l1 = Math.round(50 * (1 - l0/this.width));
          let thisColor = { h: ((this.lineHues[i] )%360) , s: 70 , l: l1 };
          this.drawPixel(15 - j, i, thisColor);
        }
      }
      this.forceRerender()
    },

    rainbowHorizontal() {
      this.renderComponent = false;
      this.hue += 2;
      for (let i = 0; i < this.width; i++) {
        let thisColor = { h: ((this.hue + i * this.scale)%360) , s: 70 , l: 50 };
        for (let j = 0; j < this.height; j++)
          this.drawPixel(i, j, thisColor);   //leds[getPixelNumber(i, j)] = thisColor;
      }
      this.forceRerender()
    },
    getRandomInt(max) {
      return Math.floor(Math.random() * Math.floor(max));
    },
    snowRoutine() {
      this.renderComponent = false;
      for (let x = 0; x < this.width; x++) {
        for (let y = 0; y < this.height - 1; y++) {
          this.drawPixelRawI2(x, y, this.getPixelColorI2(x, y + 1));
        }
      }

      for (let x = 0; x < this.width; x++) {
        // let c1 = this.getPixelColor(x, this.height - 2)
        let zeroColor = "hsl(0, 0%, 0%)";
        if (this.getPixelColorI2(x, this.height - 2) == zeroColor && (this.getRandomInt(this.scale) == 0)) {
          let n = this.getRandomInt(4);
          let color = {h: (240 - 16*n), s: (120 - 16*n),l: (90 - 16*n)};
          this.drawPixelI2(x, this.height - 1, color);
        } else
          this.drawPixelI2(x, this.height - 1, {h:0,s:0, l:0});
      }
      this.forceRerender()
    },

    color(item) {
        return "background-color: " + item;
    },
    forceRerender() {
      this.renderComponent = false;
      this.$nextTick(() => {
        this.renderComponent = true;
      });
    },

    initHues() {
        let hues = [];
        let hue=this.getRandomInt(360);
        for(let i=0;i<16;i++) {
            hue += this.getRandomInt(64) - 32;
            hues.push(hue);
        }
        return hues
    },

    initPoses() {
        let linePoses = [];
        for(let i=0;i<16;i++) {
            linePoses.push(this.getRandomInt(6));
        }
        return linePoses
    },

    initArea() {
        let result = [];
        let i,j;
        for(i=0;i<16;i++)
        for(j=0;j<16;j++) {
            let c=colorsys.stringify({r: 0, g:0, b:0});
            result.push(c);
        }
        return result;
    }
  },
  mounted() {
    this.$nextTick(function () {
      window.setInterval(() => {
          if (this.effectId == 0) {
            this.rainbowHorizontal();
          } else if (this.effectId == 1) {
            this.rainbowVertical();
          } else if (this.effectId == 2) {
            this.snowRoutine();
          } else if (this.effectId == 4) {
            this.runningLines();
          } else if (this.effectId == 5) {
            this.roundCircles();
          } else if (this.effectId == 6) {
            this.horizCircles();
          } else if (this.effectId == 7) {
            this.horizSquares();
          }
      },100);
    })
  },
  data() {
      return {
        renderComponent: true,
        area: this.initArea(),
        effectId: 4,
        width: 16,
        height: 16,
        hue: 10,
        l: 50,
        scale: 10,
        types: [
            {value: 0, text: 'Rainbow Horizontal'},
            {value: 1, text: 'Rainbow Vertical'},
            {value: 2, text: 'Snow'},
            {value: 4, text: 'Running Lines'},
            {value: 5, text: 'Round Circles'},
            {value: 6, text: 'Horizontal Circles'},
            {value: 7, text: 'Horizontal Squares'},
        ],
        blurs: [
            0,
            1,
            2,
            4,
            8,
            12,
            16,
            20,
            40
        ],
        blur: 12,
        contrasts: [
            50,
            100,
            150,
            200,
        ],
        contrast: 150,
        brightnesses: [
            0.5,
            1,
            1.5,
            2,
        ],
        brightness: 1.5,

        points: [
          {
            x: 12,
            y: 12,
            r: 3,
            hue: 240,
          },
          {
            x: 7,
            y: 7,
            r: 2,
            hue: 140,
          },
          {
            x: 3,
            y: 3,
            r: 3,
            hue: 20,
          },
          {
            x: 7,
            y: 7,
            r: 1,
            hue: 20,
          },
        ],
        x1: 7,
        y1: 7,
        r1: 3,
        linePos: 0,
        linePoses: this.initPoses(),
        lineHues: this.initHues(),
      }
  }
}
</script>

<style scoped>
  .matrix-wrap {
      float:left;
  }
  .matrix {
    display: grid;
    grid-template-columns: repeat(16, 32px);
    grid-template-rows: repeat(16, 32px);
  }
  .point {
      margin-right: 12px;
      margin-bottom: 12px;
      border-radius: 8px;
  }
</style>
