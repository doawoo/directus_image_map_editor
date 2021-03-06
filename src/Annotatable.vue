<template id="annotatable-template">
	<div class="annotatable" v-on:mousedown="beginDraw" v-on:mousemove="drawing" v-on:mouseup="drawn">
		<slot>SLOT</slot>
		<annotation v-for="annotation in annotations" :props="annotation" :input="input" :options="options"  v-bind:key="annotation">
		</annotation>
		<outline v-if="newAnnotationDrawing" :props="newAnnotation"></outline>
	</div>
</template>

<script>
import Annotation from './Annotation.vue'
import Outline from './Outline.vue'
import {randomId} from './utils.js'

import { EventBus } from './EventBus.js'

export default {
	props: {
		annotations: {},
		options: {},
		input: {default: 'input-text'}
	},
	data: function() {
		return {
			newAnnotation: null,
			newAnnotationDrawing: false,
			lock: false,
			position: null
		}
	},
	mounted() {
		EventBus.$on('labeled', (e) => this.labeled(e))
		EventBus.$on('remove', (e) => this.remove(e))
		EventBus.$on('toEdit', (e) => this.toEdit(e))
	},
	methods: {
		beginDraw: function(e) {
			if (this.lock) return;
			this.newAnnotationDrawing = true;
			this.position = e.currentTarget.getBoundingClientRect();
			this.newAnnotation = {
				x: e.clientX - this.position.left, y: e.clientY - this.position.top, width:1, height:1,
				status:'new',
				label: '',
				id: randomId()
			};
		},
		drawing: function(e) {
			if (!this.newAnnotationDrawing) return;
			this.newAnnotation.width = e.clientX - this.newAnnotation.x - this.position.left;
			this.newAnnotation.height = e.clientY - this.newAnnotation.y - this.position.top;
		},
		drawn: function(e) {
			if (this.lock) return;
			if(this.newAnnotation.width<3 || this.newAnnotation.height<3) {
				this.newAnnotationDrawing = false;
				return;
			}
			this.annotations.push(this.newAnnotation);
			this.newAnnotationDrawing = false;
			this.newAnnotation = null;
			this.lock = true;
		},
		labeled: function(msg) {
			this.lock = false;
			if (!msg) return;
			for (var i = 0; i < this.annotations.length; i++) {
				if (this.annotations[i].id == msg.id) {
					this.annotations[i].label = msg.label;
					this.annotations[i].status = 'labeled';
				}
			}
		},
		remove: function(msg) {
			this.annotations = this.annotations.filter(function(annotation){
				return annotation.id != msg.id;
			});
			this.lock = false;
		},
		toEdit: function(msg) {
			for (var i = 0; i < this.annotations.length; i++) {
				if (this.annotations[i].id == msg.id) {
					this.annotations[i].status = 'edit';
				}
			}
		},		
	},
	components: { Annotation, Outline }
};
</script>

<style>
.annotatable {
	cursor: crosshair;
	position: relative;
}
</style>
