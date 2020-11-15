<template>
  <div>
    <span v-if="this.value != null">
      <image-map :value="this.value" @input="zones => change({ zones })"/>
    </span>
  </div>
</template>

<script>
import Vue from 'vue'
  import mixin from "@directus/extension-toolkit/mixins/interface";
  import ImageMap from './ImageMap'

  export default {
    mixins: [mixin],
    components: {
      ImageMap
    },
    methods: {
      change(changes) {
        console.log( { ...this.value, ...changes });
      },
    },
    mounted() {
      this.observer = new MutationObserver((events) => {
        events.forEach(e => {
          if (e.addedNodes.length > 0) {
            const node = e.addedNodes[0]
            if (node.attributes && node.attributes.class && node.attributes.class.value == "v-card card text-background") {
              const url = node.querySelector('img').src.split('?')[0]
              this.value = {
                source: url,
                zones: [],
              }
            }
          }
        })
      })
      const config = { attributes: false, childList: true, subtree: true };
      this.observer.observe(document.querySelector(".input-single-file"), config)
    }
  }
</script>

<style lang="scss" scoped>
input {
  border-radius: var(--border-radius);
}
</style>
