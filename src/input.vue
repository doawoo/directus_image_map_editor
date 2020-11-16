<template>
  <div>
    <span v-if="this.value != null">
     <h3>MAP</h3>
      <annotatable :annotations.sync="this.value.annotations" :options="this.value.options" input="input-menu">
		    <img id="image" draggable='false' ondragstart="return false;" :src="this.value.src">
		  </annotatable>
      <output :annotations="annotations"></Output>
    </span>
  </div>
</template>

<script>
import Vue from 'vue'
  import mixin from "@directus/extension-toolkit/mixins/interface";
  import Annotatable from './Annotatable.vue'
  import Output from './Output.vue'

  import { EventBus } from './EventBus.js';

  export default {
    mixins: [mixin],
    components: {
      Annotatable
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
              this.srcUrl = url;
              this.value = {
                annotations: [],
                src: url,
                options: {"a": ["foo"]}
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
