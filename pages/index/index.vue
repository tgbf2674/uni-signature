<template>
	<view class="wrapper">
		<button @click="showCanvas = !showCanvas" type="primary" v-if="showCanvas===false">点击此处签名</button>
		<signature v-else :showCanvas="showCanvas" @closeCanvas="closeCanvas"></signature>
	</view>
</template>

<script>
	import signature from '../../components/signature.vue'
	export default {
		data() {
			return {
				//打开canvas绘制签名
				showCanvas: false,
			};
		},
		components: {
			signature
		},
		methods: {
			//隐藏canvas签名组件
			closeCanvas() {
				this.showCanvas = false;
			}
		}
	}
</script>

<style lang="scss" scoped>
	.wrapper{
		display: flex;
		justify-content: center;
		align-items: center;
	}
</style>
