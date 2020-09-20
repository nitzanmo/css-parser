<template>
  <div class="root">
    <div class="inputs">
      <div class="input-container">
        <h1>
          Classic Css Syntax
        </h1>
        <b-form-textarea :class="['css-input']" v-model="classicCss"
                         placeholder="div { background: red; max-width: 40px;..."/>
      </div>
      <div class="input-container">
        <h1>
          Object Css Syntax
        </h1>
        <b-form-textarea :class="['css-input']" v-model="objectCss"
                         placeholder="className: {  backgroundColor: 'red', maxWidth: 40,..."/>
      </div>
    </div>
    <div class="buttons">
      <b-icon-arrow-right
          @click="handleDirectionChange"
          :class="[fromClassic ? 'arrow' : 'arrow-inverted']"/>
      <b-button variant="primary" @click="convert">Convert</b-button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CssParser',
  data() {
    return {
      classicCss: ".b-icon.bi {\n" +
          "    display: inline-block;\n" +
          "    overflow: visible;\n" +
          "    vertical-align: -0.15em;\n" +
          "    width: 20px;\n" +
          "}",
      objectCss: "",
      fromClassic: sessionStorage.getItem('fromClassic') ? sessionStorage.getItem('fromClassic') === "True" : true,
    }
  },
  methods: {
    handleDirectionChange: function () {
      sessionStorage.setItem('fromClassic', this.fromClassic ? 'False' : 'True');
      this.fromClassic = !this.fromClassic;
    },
    isSinglePixelValue: function (css, index) {
      const endOfLineIndex = css.indexOf(';', index);
      // Checks if the line ends with px, and there is only one px
      return endOfLineIndex !== -1 && css.substr(endOfLineIndex - 2, 2) === 'px' &&
          css.substr(index + 1, endOfLineIndex).match(/px/g).length === 1;
    },
    isNumeric: function (n) {
      return !isNaN(parseFloat(n)) && isFinite(n);
    },
    convert: function () {
      if (this.fromClassic) {
        let objectCss = '';
        for (let i = 0; i < this.classicCss.length; i++) {
          const previousChar = this.classicCss[i - 1];
          const nextChar = this.classicCss[i + 1];
          const char = this.classicCss[i];
          const endOfLineIndex = this.classicCss.indexOf(';', i);
          
          if (char === '.' && (!previousChar || previousChar === '\n')) {
            // converts class selectors to classes by removing the .
          } else if (char === '-') {
            // removes hyphens and applies camelCase
            objectCss += nextChar.toUpperCase();
            i++;
            // converts ; to ,
          } else if (char === ';') {
            objectCss += ',';
            // inserts ' before string values when : is found
          } else if (char === ':') {
            if (nextChar === ' ') {
              objectCss += char;
            } else {
              // width:20px
              if ((this.isNumeric(nextChar) || nextChar === '-') && this.isSinglePixelValue(this.classicCss, i)) {
                objectCss += ' ' + this.classicCss.substr(i + 1, endOfLineIndex - 1 - i);
                i = endOfLineIndex - 1;
              } else {
                // color:red -> color: 'red'
                objectCss += ' \'' + this.classicCss.substr(i + 1, endOfLineIndex - 1 - i);
                objectCss += '\'';
                i = endOfLineIndex - 1;
              }
            }
          } else if (char === ' ') {
            if (nextChar === ' ') {
              objectCss += ' ';
            }
            // width: 20px
            if (previousChar === ':') {
              if (this.isNumeric(nextChar) && this.isSinglePixelValue(this.classicCss, i)) {
                objectCss += ' ' + this.classicCss.substr(i + 1, endOfLineIndex - 1 - i);
                i = endOfLineIndex - 1;
              } else {
                // color: red -> color: 'red'
                objectCss += ' \'' + this.classicCss.substr(i + 1, endOfLineIndex - 1 - i);
                objectCss += '\'';
                i = endOfLineIndex - 1;
              }
              // .bla { => bla: {
            } else if (nextChar === '{') {
              objectCss += ': ';
            }
            // } else if (char === '{') {
            //   objectCss += ': ';
          } else {
            objectCss += char;
          }
        }
        
        this.objectCss = objectCss;
        // eslint-disable-next-line no-debugger
        debugger
      } else {
        // Logic here
      }
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.inputs {
  display: flex;
  justify-content: space-evenly;
  height: 75%;
}

.css-input {
  height: 400px;
  width: 450px;
  max-height: 400px;
}

.input-container {
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  align-items: center;
}

.root {
  height: 100%
}

.arrow {
  transition: all 0.4s ease;
  height: 50px;
  width: 50px;
  cursor: pointer;
}

.arrow-inverted {
  cursor: pointer;
  transform: rotateZ(-180deg);
  transition: all 0.4s ease;
  height: 50px;
  width: 50px;
}

.buttons {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 15%;
  align-items: center;
}

</style>
