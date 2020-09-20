<template>
  <div class="root">
    <div class="inputs">
      <div class="input-container">
        <h1>
          Classic Css Syntax
        </h1>
        <b-form-textarea :class="['css-input']" v-model="classicCss"
                         @keydown.tab.prevent="tabber($event, true)"
                         placeholder="div { background: red; max-width: 40px;..."/>
      </div>
      <div class="input-container">
        <h1>
          Object Css Syntax
        </h1>
        <b-form-textarea :class="['css-input']" v-model="objectCss"
                         @keydown.tab.prevent="tabber($event, false)"
                         placeholder="className: {  backgroundColor: 'red', maxWidth: 40,..."/>
      </div>
    </div>
    <div class="buttons">
      <b-icon-arrow-right
          @click="handleDirectionChange"
          :class="[fromClassic ? 'arrow' : 'arrow-inverted']"/>
      <span id="disabled-wrapper">
        <b-button variant="primary" @click="convert">
<!--                  :disabled="!fromClassic || (fromClassic && classicCss === '' || !fromClassic && objectCss === '')">-->
          Convert
        </b-button>
        <!--        <b-tooltip target="disabled-wrapper" v-if="!fromClassic">Coming Soon</b-tooltip>-->
      </span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CssParser',
  data() {
    return {
      classicCss: "",
      objectCss: "",
      fromClassic: sessionStorage.getItem('fromClassic') ? sessionStorage.getItem('fromClassic') === "True" : true,
    }
  },
  methods: {
    handleDirectionChange() {
      sessionStorage.setItem('fromClassic', this.fromClassic ? 'False' : 'True');
      this.fromClassic = !this.fromClassic;
    },
    isSinglePixelValue(css, index) {
      const endOfLineIndex = css.indexOf(';', index);
      // Checks if the line ends with px, and there is only one px
      return endOfLineIndex !== -1 && css.substr(endOfLineIndex - 2, 2) === 'px' &&
          css.substr(index + 1, endOfLineIndex - index - 1).match(/px/g).length === 1;
    },
    isNumeric(n) {
      return !isNaN(parseFloat(n)) && isFinite(n);
    },
    convertFromClassic() {
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
              objectCss += ' ' + this.classicCss.substr(i + 1, endOfLineIndex - 3 - i);
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
              objectCss += ' ' + this.classicCss.substr(i + 1, endOfLineIndex - 3 - i);
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
        } else {
          objectCss += char;
        }
      }
      
      this.objectCss = objectCss;
    },
    convertFromObject() {
      let classicCss = '';
      for (let i = 0; i < this.objectCss.length; i++) {
        const previousChar = this.objectCss[i - 1];
        const nextChar = this.objectCss[i + 1];
        const char = this.objectCss[i];
        const endOfLineIndex =
            this.objectCss.indexOf(',', i) !== -1 ? this.objectCss.indexOf(',', i) :
                this.objectCss.indexOf('\n', i) !== -1 ? this.objectCss.indexOf('\n', i) : this.objectCss.indexOf('}', i);
        
        /// object -> .object
        if ((i === 0) || (i > 4 && (this.objectCss[i - 3] === '}' || this.objectCss[i - 2] === '}'))) {
          classicCss += '.' + char;
        } else if (char === ',') {
          // color: 'red', -> color: red;
          classicCss += ';';
        } else if (char === '\n' && nextChar === '}') {
          // color: 'red' -> color : red;
          // }               }
          if (previousChar !== ',') {
            classicCss += ';' + char;
          } else {
            classicCss += char;
          }
        } else if (char === ':') {
          // objectClass:{ -> objectClass {
          if (nextChar === '{') {
            classicCss += ' {';
            i++;
          } else if (nextChar === ' ' && this.objectCss[i + 2] === '{') {
            classicCss += ' {';
            i += 2;
          } else if (this.isNumeric(nextChar)) {
            // width:20, -> width: 20px;
            classicCss += char + " " + this.objectCss.substr(i + 1, endOfLineIndex - i - 1) + "px";
            i = endOfLineIndex - 1;
          } else if (nextChar === '\'') {
            // color:'red', -> color: red;
            classicCss += " " + this.objectCss.substr(i, endOfLineIndex - i).split('\'')[1]
            i = endOfLineIndex - 1;
          } else {
            classicCss += char;
          }
        } else if (char === ' ') {
          if (nextChar === ' ') {
            // trim double spaces
          } else if (previousChar === ':') {
            // width: 20, -> width: 20px;
            if (this.isNumeric(nextChar)) {
              // width: 20, -> width: 20px;
              classicCss += this.objectCss.substr(i, endOfLineIndex - i) + "px";
              i = endOfLineIndex - 1;
            } else if (nextChar === '\'') {
              // color:'red', -> color: red;
              classicCss += " " + this.objectCss.substr(i, endOfLineIndex - i).split('\'')[1]
              i = endOfLineIndex - 1;
            } else {
              classicCss += char;
            }
          }
        } else if (!/\s/.test(char) && char !== '}' && char === char.toUpperCase()) {
          // objectClass -> object-class
          classicCss += '-' + char.toLowerCase();
        } else {
          classicCss += char;
        }
      }
      
      this.classicCss = classicCss;
    },
    convert() {
      if (this.fromClassic) {
        this.convertFromClassic();
      } else {
        this.convertFromObject();
      }
    },
    tabber(event, isClassic) {
      const text = isClassic ? this.classicCss : this.objectCss,
          originalSelectionStart = event.target.selectionStart,
          textStart = text.slice(0, originalSelectionStart),
          textEnd = text.slice(originalSelectionStart),
          textWithTab = `${textStart}\t${textEnd}`;
      isClassic ? this.classicCss = textWithTab : this.objectCss = textWithTab;
      event.target.value = textWithTab // required to make the cursor stay in place.
      event.target.selectionEnd = event.target.selectionStart = originalSelectionStart + 1
    },
  }
  ,
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.inputs {
  display: flex;
  justify-content: space-evenly;
  height: 65%;
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
