<template>
  <transition name="el-fade-in">
    <div v-if="show">
      <div class="dropdown synonyms-conteiner is-active" :style="position">
        <div class="dropdown-menu" role="menu">
          <div class="dropdown-content">
            <div class="dropdown-item buttons are-small">
              <button class="button" @click="applyFormat('bold')">
                <span class="icon is-small">
                  <i class="fas fa-bold"></i>
                </span>
              </button>
              <button class="button" @click="applyFormat('italic')">
                <span class="icon is-small">
                  <i class="fas fa-italic"></i>
                </span>
              </button>
              <button class="button" @click="applyFormat('underline')">
                <span class="icon is-small">
                  <i class="fas fa-underline"></i>
                </span>
              </button>
            </div>

            <hr class="dropdown-divider" />

            <div class="scroll">
              <button
                v-if="isLoading"
                class="button is-primary is-outlined is-fullwidth is-large is-loading"
              >
                Loading
              </button>

              <a
                href="#"
                class="dropdown-item"
                v-for="word in words"
                :key="word"
                @click="selectSyn(word)"
                >{{ word }}</a
              >
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import axios from "axios";
import debounce from "lodash-es/debounce";

export default {
  data: function() {
    return {
      words: [],
      selection: null,
      position: {
        top: 0,
        left: 0
      },
      show: false,
      isLoading: false
    };
  },
  created() {
    // SUBSCRIBE TO `selection` EVENT
    document.addEventListener("selectionchange", this.onSelectionChange);
  },
  methods: {
    /** DEBOUNCED `selectionchange` EVENT HANDLER */
    onSelectionChange: debounce(function() {
      this.words = [];
      this.show = false;
      this.selection = document.getSelection();

      if (this.selection.rangeCount) {
        const { top, left, width } = this.selection
          .getRangeAt(0)
          .getBoundingClientRect();

        this.position = { top: top + "px", left: `${left + width / 2}px` };
      }

      const words = this.selection
        .toString()
        .trim()
        .split(" ");
      if (words.length == 1 && words[0].length) {
        this.show = true;
        this.isLoading = true;
        this.getSynonyms(words);
      }
    }, 300),
    /** REQUEST TO `datamuse` API FOR SYNONYMS */
    getSynonyms: function(wordsArray) {
      return axios
        .get(
          `https://api.datamuse.com/words?ml=${wordsArray.join("+")}&max=10`,
          { crossdomain: true }
        )
        .then(response => {
          this.isLoading = false;
          this.words = response.data.slice(0, 10).map(syn => syn.word);
        })
        .catch(error => {
          this.isLoading = false;
          console.log("datamuse error:", error);
        });
    },
    /** */
    selectSyn: function(word) {
      const range = this.selection.getRangeAt(0);
      range.deleteContents();
      range.insertNode(document.createTextNode(word));

      this.selection.removeAllRanges();
      this.selection = null;
      this.words = [];
    },
    /** APPPLY FORMAT TO SELECTED TEXT */
    applyFormat: function(name, value = null) {
      document.execCommand(name, false, value);
    }
  },
  beforeDestroy() {
    // RELEASE RESOURCES
    document.removeEventListener("selectionchange", this.onSelectionChange);
  }
};
</script>

<style lang="scss" scoped>
.synonyms-conteiner {
  z-index: 9999;
  position: fixed;
  transform: translateY(24px) translateX((190 / 2) * -1px);
  .dropdown-menu {
    .dropdown-content {
      display: flex;
      overflow: auto;
      max-height: 230px;
      flex-direction: column;
      .dropdown-divider {
        margin: 4px 0;
      }
      .buttons {
        margin: 0;
        padding: 0 8px;
        text-align: center;
        .button {
          color: $grey;
          width: 32px;
          border: none;
          height: 32px;
          padding: 0;
          font-size: 16px !important;
          margin-bottom: 0;
        }
      }
      .is-loading {
        border: none;
      }
      .scroll {
        flex: 1;
        overflow: auto;
      }
    }
  }
}
</style>
