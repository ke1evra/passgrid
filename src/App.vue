<template>
  <div id="app">
    <div class="pass-grid">
      <div class="pass-grid__header">PassGrid</div>

      <div class="pass-grid__lines">
        <input class="pass-grid__input" type="text" placeholder="Key" v-model="passPhrase" />
        <div class="pass-grid__line-container" v-for="(hash, index) in hashArray" :key="hash">
          <div class="pass-grid__line-index">{{ index + 1 }}</div>
          <div class="pass-grid__line">{{ hash }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { sha256 } from 'js-sha256';

export default {
  data() {
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
      shufflePadding: '0123012301230123012301230123012301230123'
    };
  },

  computed: {
    hash() {
      return sha256(this.passPhrase);
    },

    hashArray() {
      return this.renderHashArray(35);
    }
  },

  methods: {
    chunkString(str, length) {
      return str.match(new RegExp('.{1,' + length + '}', 'g'));
    },

    getSymbol(string) {
      const replaceIndex = this.symbolIndex;
      const index = parseInt(string[replaceIndex], 16);

      return this.symbols[index];
    },

    getUpperChar(string) {
      const replaceIndex = this.upperCharIndex;
      const index = parseInt(string[replaceIndex], 16);
      const padding = parseInt(string[0], 16);

      return this.upperChars[index + padding];
    },

    getLowerChar(string) {
      const replaceIndex = this.lowerCharIndex;
      const index = parseInt(string[replaceIndex], 16);
      const padding = parseInt(string[0], 16);

      return this.lowerChars[index + padding];
    },

    getDigit(string) {
      const replaceIndex = this.digitIndex;
      const index = parseInt(string[replaceIndex], 16);
      const padding = parseInt(string[1], 16);

      return this.digits[index + padding];
    },

    renderChunk(chunk) {
      const result = chunk.split('');

      const padding = parseInt(chunk[0], 16);

      result[this.shufflePadding[this.digitIndex + padding]] = this.getDigit(chunk);
      result[this.shufflePadding[this.lowerCharIndex + padding]] = this.getLowerChar(chunk);
      result[this.shufflePadding[this.upperCharIndex + padding]] = this.getUpperChar(chunk);
      result[this.shufflePadding[this.symbolIndex + padding]] = this.getSymbol(chunk);

      return result.join('');
    },

    renderHash(string) {
      const chunks = this.chunkString(string, this.chunkSize);
      const normalizedChunks = chunks.map(this.renderChunk);

      return normalizedChunks.join('');
    },

    renderHashArray(length) {
      const array = [];
      let count = 0;

      while (count < length) {
        const lastHash = array.length ? array[array.length - 1] : this.hash;

        array.push(this.renderHash(sha256(lastHash)));
        count++;
      }

      return array;
    }
  }
};
</script>
<style lang="scss">
body {
  background-color: $--color-gray-8;
}

.pass-grid {
  color: $--color-gray-3;
  font-family: ui-monospace, monospace;

  display: flex;
  align-items: center;
  flex-direction: column;

  &__input {
    width: 100%;
  }

  &__header {
    font-size: 2rem;
    padding: 0.25rem;
  }

  &__lines {
    margin-top: 1rem;
  }

  &__line-index {
    padding-right: 0.25rem;
    text-align: right;
    width: 1.5rem;
    color: $--color-gray-7;
    -webkit-touch-callout: none; /* iOS Safari */
    -webkit-user-select: none; /* Safari */
    -khtml-user-select: none; /* Konqueror HTML */
    -moz-user-select: none; /* Old versions of Firefox */
    -ms-user-select: none; /* Internet Explorer/Edge */
    user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome, Edge, Opera and Firefox */
  }

  &__line-container {
    display: flex;
    flex-direction: row;
  }

  &__line {
    display: block;
  }
}
</style>
