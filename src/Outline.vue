<template>
	<div class="outline" v-bind:style="size"></div>
</template>

<script>
import {sizeToStyle} from './utils'

export default {
	props: ['props'],
	computed: {
		size: {
			cache: false,
			get: function(){
				return sizeToStyle(this.props);
			},
		}
	}
}
</script>

<style>
.outline {
	outline: 1px dashed rgba(255,0,255,1.0);
	border: 1px dashed rgba(255,0,255,1.0);
	position: absolute;
}
</style>