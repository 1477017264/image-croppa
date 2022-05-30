<template>
	<div id="app">
		<image-croppa @generateImageBlob="generateImageBlob"></image-croppa>
	</div>
</template>

<script>
	import ImageCroppa from './components/ImageCroppa'

	export default {
		name: 'App',

		methods: {
			generateImageBlob(blob) {

				var url = URL.createObjectURL(blob)
				var a = document.createElement('a');
				a.download = 'filename';
				a.href = url;
				a.click();
				URL.revokeObjectURL(url);
			}
		},

		components: {
			ImageCroppa
		}
	}
</script>

<style>
	#app {
		font-family: 'Avenir', Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #2c3e50;
		margin-top: 60px;
	}

	.el-message-box {
		width: auto;
		border-radius: 8px;
	}

	.el-message-box .el-message-box__container {
		margin-top: 6px;
		font-size: 15px;
	}

	.el-message-box__btns .el-button--small.el-button--primary {
		border-radius: 8px;
		width: 80px;
		margin-bottom: 6px;
	}
</style>
