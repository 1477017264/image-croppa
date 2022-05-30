<template>

	<div class="image-croppa">
		<el-row class="croppa-panel">
			<el-col :span="11" style="width: auto;margin:10px auto;">
				<div class="box-title" style="">️️<span>🎆 上传图片 🎆</span></div>
				<div class="center-block">
					<div class="croppa-wrapper-control"
						:style="{width:controlMaxWidth+'px',height:controlMaxHeight+'px'}">
						<croppa ref="croppa" v-model="myCroppa" :width="width" :height="height"
							placeholder="点击或拖拽一张图片到这里" :zoom-speed="1" :quality="quality" :prevent-white-space="true"
							@new-image-drawn="updatePreviewImage" @image-remove="imageRemove"></croppa>
					</div>
				</div>

			</el-col>
			<el-col class="update-view-control" :span="2">

			</el-col>
			<el-col :span="11" style="width: auto;margin:10px auto;">
				<div class="box-title">️<span>🎑 效果预览 🎑</span></div>
				<div class="center-block">
					<div class="croppa-wrapper-control"
						:style="{width:controlMaxWidth+'px',height:controlMaxHeight+'px'}">
						<div class="preview-image-box" :style="{width:width+'px',height:height+'px'}">
							<img :src="previewSrc" alt="预览图" width="100%" height="100%">
						</div>
					</div>
				</div>
			</el-col>
		</el-row>


		<div class="croppa-control-bar">
			<el-form>
				<el-col :span="4">
					<el-form-item label="常用尺寸" align="right" :label-width="labelWidth">
						<el-select v-model="currentSizeIndex">
							<el-option v-for="(size, index) in sizeList" :key="index" :label="size" :value="index">
							</el-option>
						</el-select>
					</el-form-item>
				</el-col>

				<el-col :span="7">
					<el-form-item v-show="currentSizeIndex===0" label="自定尺寸" :label-width="labelWidth">
						<el-col :span="10">
							<el-input type="number" v-model.number="customWidth"></el-input>
						</el-col>
						<el-col :span="4" align="center">×</el-col>
						<el-col :span="10">
							<el-input type="number" v-model.number="customHeight"></el-input>
						</el-col>
					</el-form-item>
				</el-col>

				<el-col :span="4">
					<el-form-item label="图片类型" :label-width="labelWidth">
						<el-select v-model="curMIMEtypeIndex" @change="updatePreviewImage">
							<el-option v-for="(type, index) in MIMEtypes" :key="index" :label="type" :value="index">
							</el-option>
						</el-select>
					</el-form-item>
				</el-col>

				<el-col :span="4">
					<el-form-item v-if="!curMIMEtypeIndex" label="压缩比例" :label-width="labelWidth">
						<el-select v-model="curCompressionRateIndex" @change="updatePreviewImage">
							<el-option v-for="(rate, index) in compressionRates" :key="index" :label="rate"
								:value="index"></el-option>
						</el-select>
					</el-form-item>
				</el-col>

				<el-col :span="5" align="center">

					<el-button type="primary" @click="cut" size="medium">💾 裁剪并下载 💾</el-button>

				</el-col>

			</el-form>
		</div>
		<div class="footer"><span><a href="https://dadio.cc" target="_blank">&copy; 2022 - DaDio</a></span></div>
	</div>

</template>

<script>
	/*
		该组件强制依赖于vue、Ement-ui、vue-croppa
		需在全局main.js注册croppa，步骤如下：
		
		import Croppa from 'vue-croppa'
		import 'vue-croppa/dist/vue-croppa.css'
		Vue.use(Croppa)
		
		
	 */
	export default {

		data() {
			return {
				labelWidth: '80px',

				//裁剪框显示的最大宽高默认值
				controlMaxWidth: 400,
				controlMaxHeight: 400,

				myCroppa: {},


				//常用海报可选尺寸
				sizeList: ['自定义宽高', '200x200', '200x300', '300x200', '300x300', '400x300', '300x400', '600x600',
					'1280x640', '640x1280', '1920x1080', '1080x1920'
				],
				//当前选中的尺寸索引值	
				currentSizeIndex: 0,

				//自定义宽高
				customWidth: 600,
				customHeight: 600,

				//可生成的图片类型
				MIMEtypes: ['image/jpeg', 'image/png'],
				//可选压缩率	
				compressionRates: ['默认', '0.9', '0.8', '0.7', '0.6', '0.5', '0.4', '0.3', '0.2', '0.1'],
				//当前选中的图片类型索引值
				curMIMEtypeIndex: 0,
				//当前选中的压缩率索引值
				curCompressionRateIndex: 0,

				//预览图路径
				previewSrc: '',

				generatingFlag: false,
			}
		},

		computed: {
			sizeInfo() {
				let selectedSizeText = this.sizeList[this.currentSizeIndex];

				if (selectedSizeText === '自定义宽高') {
					return [this.customWidth || 5, this.customHeight || 5];
				} else {

					return selectedSizeText.split("x");
				}
			},

			//所选生成尺寸宽度
			w() {
				return parseInt(this.sizeInfo[0], 10);
			},

			//所选生成尺寸高度
			h() {
				return parseInt(this.sizeInfo[1], 10);
			},

			//裁剪框大小 相对于 生成尺寸大小 的缩放度
			scale() {
				if (this.w <= this.controlMaxWidth && this.h <= this.controlMaxHeight) {
					return 1;
				} else {
					//以较大的边为准
					if (this.w >= this.h) {
						return this.controlMaxWidth / this.w;

					} else {
						return this.controlMaxHeight / this.h;
					}
				}
			},

			//裁剪框实际显示宽度
			width() {
				let tmp = this.w * this.scale;
				return tmp > this.controlMaxWidth ? this.controlMaxWidth : tmp;;
			},

			//裁剪框实际显示高度
			height() {
				let tmp = this.h * this.scale;
				return tmp > this.controlMaxHeight ? this.controlMaxHeight : tmp;;
			},

			//croppa的quality属性值，标识输出的图片尺寸大小相对于裁剪框大小的倍数
			//有如下计算关系：
			//		 outputWidth = width * quality,
			//		 outputHeight = height * quality
			quality() {
				return this.w / this.width;
			}
		},

		watch: {
			//监听myCroppa位置变化，实时更新预览图
			'myCroppa.imgData': {
				handler() {
					if (!this.generatingFlag) {

						this.updatePreviewImage();
					}
				},
				deep: true
			}
		},

		methods: {

			cut() {

				if (!this.previewSrc) {

					this.$alert('请先选择一张图片进行裁剪！', {
						type: 'warning'
					});

				} else {
					this.generateImage();
				}
			},

			generateImage(callback) {

				//测试发现，压缩率貌似只和jpg图片类型有关
				if (!this.curMIMEtypeIndex && this.curCompressionRateIndex) {

					var compressionRate = parseFloat(this.compressionRates[this.curCompressionRateIndex]);
				} else {
					var compressionRate = 1;
				}

				this.myCroppa.generateBlob((blob) => {

					if (typeof callback === "function") {
						callback(blob);
					} else {

						this.$emit("generateImageBlob", blob);
						this.myCroppa.remove()
					}

				}, this.MIMEtypes[this.curMIMEtypeIndex], compressionRate)

			},


			//更新预览图
			updatePreviewImage() {
				this.generatingFlag = true;

				this.generateImage((blob) => {
					this.generatingFlag = false;

					if (blob) {
						var url = URL.createObjectURL(blob)
						this.previewSrc = url;
					}
				});
			},

			imageRemove() {
				this.previewSrc = '';
			}

		},
	}
</script>

<style>
	#app {
		margin-top: 38px !important;

	}

	.icon.icon-remove {
		width: 22px;
		height: 22px;
		filter: drop-shadow(0px 0px 2px rgba(0, 0, 0, 0.5)) !important;
	}

	.image-croppa {
		display: flex;
		flex-direction: column;
		flex-wrap: nowrap;
		align-content: center;
		justify-content: center;
		align-items: center;
		width: 100%;
	}

	.image-croppa .croppa-wrapper-control,
	.image-croppa .update-view-control {
		text-align: center;
		display: table-cell;
		vertical-align: middle;
		line-height: 0;
		}
	
	.image-croppa .croppa-wrapper-control{
		width: 360px !important;
		height: 360px !important;
	}

	.image-croppa .update-view-control {
		height: 100%;
	}

	.croppa-panel .el-col {
		position: relative;
		text-align: center;
	}

	.croppa-wrapper-control .croppa-container {
		max-width: 360px ;
		max-height: 360px ;
		border-radius: 8px;
		background: rgba(0, 0, 0, 0.05);
	}

	.croppa-wrapper-control .croppa-container canvas {
		/* 		width: 360px;
		height: 360px; */
		max-width: 360px;
		max-height: 360px;
		border-radius: 8px;
	}

	.center-block .croppa-wrapper-control .preview-image-box {
		max-height: 360px;
		max-width: 360px;
		border-radius: 8px;
		margin: 0 auto;
	}

	.center-block .croppa-wrapper-control .preview-image-box img {
		border-radius: 8px;
	}

	/* 	.el-col-10 .el-input__inner{
		width: 90px;
		margin-left: ;
	} */

	.croppa-panel.el-row {
		display: flex;
		flex-direction: column;
		align-content: center;
		justify-content: center;
		align-items: center;
	}
	
	.image-croppa .croppa-control-bar .el-form {
		clear: left;
		margin: 10px;
		text-align: center;
		display: flex !important;
		flex-direction: column;
		flex-wrap: nowrap;
		align-content: center;
		justify-content: center;
		align-items: center;
	}

	.image-croppa .croppa-control-bar .el-form-item__label {
		text-align: left;
	}

	.image-croppa .croppa-control-bar .el-form .el-col-4 {
		width: auto;
		margin: 10px;
	}
	
	.image-croppa .croppa-control-bar .el-form .el-col-5{
		width: auto;
		margin: 10px;
	}
	
	.image-croppa .croppa-control-bar .el-form .el-col-7 {
		width: auto;
		margin: 10px;
	}

	.image-croppa .croppa-control-bar .el-form .el-col-5 .el-button {
		width: 280px;
		border-radius: 8px;
		height: 42px;
	}

	.image-croppa .croppa-control-bar .el-form .el-col-4 .el-form-item__content {
		width: 200px;
		text-align: center;
	}

	.image-croppa .el-input__inner {
		border-radius: 8px;
	}

	.el-select-dropdown__item.hover,
	.el-select-dropdown__item {
		transition: 0.2s ease-in-out;
	}

	.el-select-dropdown__item.hover,
	.el-select-dropdown__item:hover {
		border-radius: 8px;
		margin: 0 3px;
		font-weight: bold;
		font-size: 110%;
	}

	.image-croppa .croppa-control-bar .el-form .el-col-7 .el-form-item__content {
		width: 200px;
		text-align: center;

	}

	.image-croppa .croppa-control-bar .el-form .el-col-7 .el-form-item__content .el-col.el-col-4 {
		float: left;
		margin: 0 5px 0 14px;
		font-weight: bold;
		font-family: sans-serif;
	}

	.image-croppa .croppa-control-bar .el-form .el-col-7 .el-form-item__content .el-input__inner {
		padding: 0;
		padding-left: 15px;
		width: 90px;
	}

	.el-select-dropdown.el-popper {
		border-radius: 8px;
	}

	.croppa-panel .center-block {
		/* display: inline-block; */
		margin: 26px auto;
	}

	.image-croppa .box-title {
		position: absolute;
		top: -49px;
		text-align: center;
		border-radius: 8px;
		width: 100%;
		box-sizing: inherit;
	}

	.image-croppa .box-title span {
		margin: auto;
		text-align: center;
		background: rgba(0, 0, 0, 0.05);
		display: block;
		border-radius: 8px;
		padding: 10px;
		width: 360px;
		box-sizing: inherit;
		font-weight: bold;
	}

	.image-croppa .preview-image-box {
		display: block;
		transition: all 0.3s;
		position: relative;
		font-size: 0;
		-ms-flex-item-align: start;
		align-self: flex-start;
		background-color: rgba(0, 0, 0, 0.05);
	}

	.image-croppa .croppa-control-bar {
		clear: left;
		margin: 10px;
		text-align: center;
	}

	.image-croppa canvas {
		display: block;
	}

	.image-croppa .el-form-item {
		margin-bottom: 0;
	}

	.el-message-box {
		width: auto;
		border-radius: 8px;
	}

	.el-message-box__container {
		margin-top: 6px;
		font-size: 15px;
	}

	.el-button--small.el-button--primary {
		border-radius: 8px;
		width: 80px;
		margin-bottom: 6px;
	}

	.footer {
		background: rgba(0, 0, 0, 0.05);
		border-radius: 8px;
		width: 100%;
		height: 50px;
		display: flex;
		flex-direction: column;
		flex-wrap: nowrap;
		align-content: center;
		justify-content: center;
		align-items: center;
	}
	
	.footer a{
		font-family: sans-serif;
		font-size: 18px;
		font-weight: bold;
		color: #2c3e50;
		text-decoration: none;
	}
	
	@media screen and (min-width: 926px) {
		.image-croppa {
			display: flex;
			/* flex-direction: column;
			flex-wrap: nowrap;
			align-content: center;
			justify-content: center;
			align-items: center;
			width: 100%; */
		}
		
		.croppa-panel .el-col {
			position: relative;
			text-align: center;
			margin: 20px 30px;
		}
		
		.croppa-panel.el-row {
			display: flex;
			flex-direction: row;
			align-content: center;
			justify-content: center;
			align-items: center;
		}
		
		.croppa-wrapper-control .croppa-container {
			max-width: 400px;
			max-height: 400px;
			border-radius: 8px;
		}
		
		.croppa-wrapper-control .croppa-container canvas {
			/* 		width: 360px;
			height: 360px; */
			max-width: 400px;
			max-height: 400px;
			border-radius: 8px;
		}
		
		.center-block .croppa-wrapper-control .preview-image-box {
			max-height: 400px;
			max-width: 400px;
			border-radius: 8px;
			margin: 0 auto;
		}
		
		.image-croppa .box-title span {
			width: 400px;
		}
		
		.footer{
			margin-top: 20px;
		}
		
	}
	
	/* 滚动条 */
	::-webkit-scrollbar-thumb {
		background-color: #409EFF;
		background-image: -webkit-linear-gradient(45deg,
				rgba(255, 255, 255, 0.4) 25%,
				transparent 25%,
				transparent 50%,
				rgba(255, 255, 255, 0.4) 50%,
				rgba(255, 255, 255, 0.4) 75%,
				transparent 75%,
				transparent);
		border-radius: 3em;
	}
	
	::-webkit-scrollbar-track {
		background-color: #ffffff;
		border-radius: 3em;
	}
	
	::-webkit-scrollbar {
		width: 6px;
		height: 10px;
	}
	
</style>
