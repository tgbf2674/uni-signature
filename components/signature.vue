<template>
	<view class="signature-box">

		<!-- 签名 -->
		<view class="signature" v-show="showCanvas">
			<canvas class="mycanvas" canvas-id="mycanvas" @touchstart="touchstart" @touchmove="touchmove"
				@touchend="touchend"></canvas>
			<view class="footer">
				<button @click="finish" type="primary" :plain="true" :ripple="true"
					ripple-bg-color="#909399">保存</button>
				<button @click="clear" type="warning" :plain="true" :ripple="true" ripple-bg-color="#909399">清除</button>
				<button @click="close" type="error" :plain="true" :ripple="true" ripple-bg-color="#909399">关闭</button>
			</view>
		</view>


		<!-- 签完名后生成的图片 -->
		<view v-show="SignatureImg" class="SignatureImg">
			<image :src="SignatureImg" mode=""></image>
		</view>

		<!-- 清除签完名后生成的图片 -->
		<u-button v-show="SignatureImg" @click="obliterate" type="error" :plain="true" :ripple="true"
			ripple-bg-color="#909399" size="medium">清除签名</u-button>
	</view>

</template>

<script>
let x = 20;
let y = 20;
export default {
	data() {
		return {
			//绘图图像
			ctx: '', 
			//路径点集合
			points: [], 
			//签名图片
			SignatureImg: ''
		};
	},
	props: ['showCanvas'],
	methods: {
		//清除签名的图片
		obliterate() {
			if (this.SignatureImg) {
				this.SignatureImg = '';
			}
			this.close();
		},
		//关闭并清空画布
		close() {
			this.$emit('closeCanvas');
			this.clear();
		},
		//创建并显示画布
		createCanvas() {
			this.ctx = uni.createCanvasContext('mycanvas', this); //创建绘图对象
			//设置画笔样式
			this.ctx.lineWidth = 4;
			this.ctx.lineCap = 'round';
			this.ctx.lineJoin = 'round';
		},
		//触摸开始，获取到起点
		touchstart(e) {
			let startX = e.changedTouches[0].x;
			let startY = e.changedTouches[0].y;
			let startPoint = { X: startX, Y: startY };
			this.points.push(startPoint);
			//每次触摸开始，开启新的路径
			this.ctx.beginPath();
		},
		//触摸移动，获取到路径点
		touchmove(e) {
			let moveX = e.changedTouches[0].x;
			let moveY = e.changedTouches[0].y;
			let movePoint = { X: moveX, Y: moveY };
			this.points.push(movePoint); //存点
			let len = this.points.length;
			if (len >= 2) {
				this.draw(); //绘制路径
			}
		},
		// 触摸结束，将未绘制的点清空防止对后续路径产生干扰
		touchend() {
			this.points = [];
		},
		//绘制笔迹
		draw() {
			let point1 = this.points[0];
			let point2 = this.points[1];
			this.points.shift();
			this.ctx.moveTo(point1.X, point1.Y);
			this.ctx.lineTo(point2.X, point2.Y);
			this.ctx.stroke();
			this.ctx.draw(true);
		},
		//清空画布
		clear() {
			let that = this;
			uni.getSystemInfo({
				success: function(res) {
					let canvasw = res.windowWidth;
					let canvash = res.windowHeight;
					that.ctx.clearRect(0, 0, canvasw, canvash);
					that.ctx.draw(true);
				}
			});
		},
		//完成绘画并保存到本地
		finish() {
			let that = this
			uni.canvasToTempFilePath(
				{
					canvasId: 'mycanvas',
					success: function(res) {
						console.log(res);
						that.SignatureImg = res.tempFilePath;
						that.$emit('closeCanvas');
						that.close();
					}
				},that
			);
		}
	},
	mounted() {
		this.createCanvas();
	}
};
</script>


<style lang="scss" scoped>
	.signature-box {
		display: flex;
		flex-direction: column;
		align-items: center;
		background: #fff;

		//签名模块
		.signature {
			position: fixed;
			top: 10px;
			left: 2%;
			z-index: 999;
			width: 96%;

			//canvas
			.mycanvas {
				width: 100%;
				height: calc(100vh - 200upx);
				background-color: #ececec;
			}

			//底部按钮
			.footer {
				font-size: 14px;
				height: 150upx;
				display: flex;
				justify-content: space-around;
				align-items: center;
			}
		}

		//生成的图片
		.SignatureImg {
			image {
				width: 750rpx;
				height: 750rpx;
			}
		}
	}
</style>
