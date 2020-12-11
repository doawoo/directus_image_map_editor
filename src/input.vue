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

  async function fetchPlaces() {
    const jsonData = await fetch('/_/items/place', {method: 'GET', credentials: 'include'})
    .then((resp) => resp.json())

    console.log(`Fetched places!`)

    return jsonData.data.map((item) => {
      return {id: item.id, name: item.name}
    })
  }

  export default {
    mixins: [mixin],
    components: {
      Annotatable
    },
    methods: {
      emitValue() {
        console.log("emitValue", this.value.annotations)
        this.$emit("input", this.value.annotations)
      },
      setImage(newUrl) {
        this.srcUrl = newUrl;
        console.log("Set image source URL", this.srcUrl);
      },
      fetchImage() {
        const node = document.getElementsByClassName("v-card card text-background")[0]
        if (node && node.querySelector('img')) {
          const image = node.querySelector('img').src.split('?')[0]
          this.setImage(image)
        } else {
          console.log("No image yet! Going to watch for mutators to detect it...")
          this.beginListenImage()
        }
      },
      beginListenImage() {
        this.observer = new MutationObserver((events) => {
          events.forEach(e => {
            if (e.addedNodes.length > 0) {
              const node = e.addedNodes[0]
              if (node.attributes && node.attributes.class && node.attributes.class.value == "v-card card text-background") {
                const url = node.querySelector('img').src.split('?')[0]
                this.setImage(url)
              }
            }
          })
        })
        const config = { attributes: false, childList: true, subtree: true }
        this.observer.observe(document.querySelector(".input-single-file"), config)
      }
    },
    mounted() {
      setTimeout(() => this.fetchImage(), 1000)
    }
  }
</script>

<style lang="scss" scoped>
input {
  border-radius: var(--border-radius);
}
</style>
