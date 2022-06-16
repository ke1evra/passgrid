<template>
  <div id="app">
    <input type="text" placeholder="Фраза" v-model="passPhrase">
    <div id="result">{{hash}}</div>
    <hr>
    <div class="" v-for="hash in hashArray" :key="hash">{{hash}}</div>
  </div>
</template>

<script>
import { sha256 } from 'js-sha256';

export default {
  data(){
    return {
      chunkSize: 4,
      symbolIndex: 3,
      upperCharIndex: 2,
      lowerCharIndex: 1,
      digitIndex: 0,
      passPhrase: '',
      symbols: '!#$%&*+-.?@^_`|~',
      upperChars: 'ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOP',
      lowerChars: 'abcdefghijklmnopqrstuvwxyzabcdefghijklmnop',
      digits: '012345678901234567890123456789012345678901',
      padding: '0123012301230123012301230123012301230123'
    }
  },

  computed: {
    hash(){
      return sha256(this.passPhrase);
    },

    hashArray(){
      return this.renderHashArray(100);
    }
  },

  methods: {
    chunkString(str, length) {
      return str.match(new RegExp('.{1,' + length + '}', 'g'));
    },

    getSymbol(string){
      const replaceIndex = this.symbolIndex;
      const index = parseInt(string[replaceIndex], 16)

      return this.symbols[index];
    },

    getUpperChar(string){
      const replaceIndex = this.upperCharIndex;
      const index = parseInt(string[replaceIndex], 16);
      const padding = parseInt(string[0], 16);

      return this.upperChars[index + padding];
    },

    getLowerChar(string){
      const replaceIndex = this.lowerCharIndex;
      const index = parseInt(string[replaceIndex], 16);
      const padding = parseInt(string[0], 16);

      return this.lowerChars[index + padding];
    },

    getDigit(string){
      const replaceIndex = this.digitIndex;
      const index = parseInt(string[replaceIndex], 16);
      const padding = parseInt(string[1], 16);

      return this.digits[index + padding];
    },

    renderChunk(chunk){
      const result = chunk.split('');
      result[this.digitIndex] = this.getDigit(chunk)
      result[this.lowerCharIndex] = this.getLowerChar(chunk);
      result[this.upperCharIndex] = this.getUpperChar(chunk);
      result[this.symbolIndex] = this.getSymbol(chunk);

      return result.join('');
    },

    renderHash(string){
      const chunks = this.chunkString(string, this.chunkSize);
      const normalizedChunks = chunks.map(this.renderChunk);

      return normalizedChunks.join('');
    },

    renderHashArray(length){
      const array = [];
      let count = 0;

      while(count < length){
        const lastHash = array.length ? array[array.length - 1] : this.hash;

        array.push(this.renderHash(sha256(lastHash)))
        count++
      }

      return array;
    }
  }
}
</script>
<style lang="scss">
  body {
    background-color: $--color-gray-8;
    color: $--color-gray-3;
    font-family: ui-monospace, monospace;
  }
</style>
