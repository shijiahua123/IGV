<template>
	<view class="gradient-slider" @tap="onClick" :style="{background: sliderBackground, borderRadius: '25px', border: '2px solid'+ borderColor}">
		<!-- 蓝色进度条 -->
		<view class="progress-bar" :style="[barStyle, {height: height + 'px', backgroundColor: '#0F40F5'}]"></view>
		<view class="gradient-slider-gap" :style="[barStyle, {height: height + 'px'}]">
			<view class="gradient-slider-wrap" @touchstart="onTouchStart" @touchmove="onTouchMove"
				@touchend="onTouchEnd" @touchcancel="onTouchEnd">
				<view :style="[{height: blockHeight + 'px',
						width: blockWidth + 'px',
						borderRadius: '50%', border: '2px solid'+ borderColor,
						background: '#fff'}]">
				</view>
			</view>
		</view>
	</view>
</template>
<script>
	/**
	 * 支持颜色渐变的slider 滑块选择器
	 * @property {Number | String} value 滑块默认值（默认0）
	 * @property {Number | String} min 最小值（默认0）
	 * @property {Number | String} max 最大值（默认100）
	 * @property {Number | String} step 步长（默认1）
	 * @property {Number | String} blockWidth 滑块宽度（20）
	 * @property {Number | String} blockHeight 滑块高度（40）
	 * @property {Number | String} height 滑块条高度，单位px（默认20）
	 * @property {String} blockBackground 滑块颜色（默认为空时支持滑块随背景色变化）
	 * @property {String} startColor 渐变起始颜色（默认#0F40F5
	 * @property {String} endColor 渐变结束颜色（默认#FFFFFF）
	 * @property {Number} border 滑动条和滑块的边框大小
	 * @property {String} borderColor 滑动条和滑块的边框颜色 （默认#333333）
	 * @property {Number} borderRadius 滑动条和滑块的圆角 （默认 8px)
	 * @event {Function} start 滑动触发
	 * @event {Function} moving 正在滑动中
	 * @event {Function} end 滑动结束
	 * @event {Function} onColorChanged 当前颜色值
	 * @example <gradient-slider v-model="value" />
	 */
	export default {
		name: "custom-slider",
		props: {
			// 当前进度百分比值，范围0-100
			value: {
				type: [Number, String],
				default: 0
			},
			// 滑块宽度，单位px
			blockWidth: {
				type: [Number, String],
				default: 20
			},
			// 滑块高度，单位px
			blockHeight: {
				type: [Number, String],
				default: 20
			},
			// 自定义滑块颜色
			blockBackground: {
				type: [String],
				default:''
			},
			// 最小值
			min: {
				type: [Number, String],
				default: 0
			},
			// 最大值
			max: {
				type: [Number, String],
				default: 100
			},
			// 步进值
			step: {
				type: [Number, String],
				default: 1
			},
			// 滑块条高度，单位px
			height: {
				type: [Number, String],
				default: 15
			},
			// 渐变开始颜色 #0F40F5
			startColor: {
				type: [String],
				default:'#0F40F5'
			},
			// 渐变结束颜色 #FFFFFF
			endColor: {
				type: [String],
				default:'#FFFFFF'
			},
			// 进度条和滑块的边框粗细
			border: {
				type: [Number],
				default: 1
			},
			borderColor: {
				type: [String],
				default:'#0F40F5'
			},
			// 进度条和滑块的边框圆角
			borderRadius: {
				type: [Number],
				default: 50
			}
		},	
		data() {
			return {
				barStyle: {},
				blockColor: '',
				status: 'end',
				sliderRect: {
					left: 0,
					width: 0
				},
				newValue: 0,
				sliderBackground: ''
			}
		},
		watch: {
			value(n) {
				// 只有在非滑动状态时，才可以通过value更新滑块值，这里监听，是为了让用户触发
				if (this.status == 'end') {
					this.updateValue(this.value, false)
				}
			}
		},
		created() {
			this.updateValue(this.value, false);
			// 移除渐变背景，使用纯色背景
			this.sliderBackground = '#fff'
		},
		mounted() {
			// 获取滑块条的尺寸信息
			this.$nextTick(() => {
				const query = uni.createSelectorQuery().in(this);
				query.select('.gradient-slider').boundingClientRect(rect => {
					if (rect) {
						this.sliderRect = rect;
					}
				}).exec();
			});
		},
		methods: {
			onTouchStart() {
				// 标示当前的状态为开始触摸滑动
				this.status = 'start';
			},
			onTouchMove(event) {
				// 连续触摸的过程会一直触发本方法，但只有手指触发且移动了才被认为是拖动了，才发出事件
				// 触摸后第一次移动已经将status设置为moving状态，故触摸第二次移动不会触发本事件
				if (this.status == 'start') this.$emit('start');
				
				// 检查sliderRect是否有效
				if (!this.sliderRect || !this.sliderRect.width) {
					// console.warn('sliderRect not ready, re-fetching...');
					this.$nextTick(() => {
						const query = uni.createSelectorQuery().in(this);
						query.select('.gradient-slider').boundingClientRect(rect => {
							if (rect) {
								this.sliderRect = rect;
							}
						}).exec();
					});
					return;
				}
				
				let touches = event.touches[0];
				// 滑块的左边不一定跟屏幕左边接壤，所以需要减去最外层父元素的左边值
				let distanceX = touches.clientX - this.sliderRect.left;
				// 确保distanceX在有效范围内
				distanceX = Math.max(0, Math.min(distanceX, this.sliderRect.width));
				// 获得移动距离对整个滑块的百分比值，此为带有多位小数的值，不能用此更新视图
				// 否则造成通信阻塞，需要每改变一个step值时修改一次视图
				this.newValue = (distanceX / this.sliderRect.width) * 100;
				// console.log('onTouchMove:', { 
				// 	clientX: touches.clientX, 
				// 	sliderLeft: this.sliderRect.left, 
				// 	sliderWidth: this.sliderRect.width,
				// 	distanceX, 
				// 	newValue: this.newValue 
				// });
				this.status = 'moving';
				// 发出moving事件
				this.$emit('moving');
				this.updateValue(this.newValue, true);
			},
			onTouchEnd() {
				if (this.status === 'moving') {
					this.updateValue(this.newValue, false);
					this.$emit('end');
				}
				this.status = 'end';
			},
			updateValue(value, drag) {
				// 去掉小数部分，同时也是对step步进的处理
				const width = this.format(value);
				// console.log('updateValue:', { value, width, drag, status: this.status });
				// 不允许滑动的值超过max最大值，百分比也不能超过100
				if (width > this.max || width > 100) return;
				// 设置移动的百分比值
				let barStyle = {
					width: width + '%'
				};
				// 移动期间无需过渡动画
				if (drag == true) {
					barStyle.transition = 'none';
				} else {
					// 非移动期间，删掉对过渡为空的声明，让css中的声明起效
					delete barStyle.transition;
				}
				// 修改value值
				this.$emit('input', width);
				this.barStyle = barStyle;
				// 使用固定的蓝色滑块
				this.blockColor = '#0F40F5';
				this.$emit('onColorChanged', this.blockColor);
				// this.$emit('onValueChanged', this.value);
				
			},
			format(value) {
				// 将小数变成整数，为了减少对视图的更新，造成视图层与逻辑层的阻塞
				const min = Number(this.min);
				const max = Number(this.max);
				const step = Number(this.step);
				const clampedValue = Math.max(min, Math.min(value, max));
				const result = Math.round(clampedValue / step) * step;
				// console.log('format:', { value, min, max, step, clampedValue, result });
				return result;
			},
			onClick(event) {
				// 直接点击滑块的情况，计算方式与onTouchMove方法相同
				const distanceX = event.detail.x - this.sliderRect.left;
				// 确保distanceX在有效范围内
				const clampedDistanceX = Math.max(0, Math.min(distanceX, this.sliderRect.width));
				const value = (clampedDistanceX / this.sliderRect.width) * 100;
				this.updateValue(value, false);
			},
			calculataColor(currentValue) {
				// 使用传入的currentValue参数，如果没有传入则使用this.value
				const value = currentValue !== undefined ? currentValue : this.value;
				
				let startColor = this.startColor.replace("#","")
				let endColor = this.endColor.replace("#","")
				
				let startR = parseInt(startColor.substring(0,2), 16) 
				let endR = parseInt(endColor.substring(0,2), 16)
				let r = Math.round((endR - startR)*value/100 + startR).toString(16)
				if(r.length == 1) {
					r = "0"+r
				}
				
				let startG = parseInt(startColor.substring(2,4), 16)
				let endG = parseInt(endColor.substring(2,4), 16)
				let g = Math.round((endG - startG)*value/100 + startG).toString(16)
				if(g.length == 1) {
					g = "0"+g
				}
				let startB = parseInt(startColor.substring(4,6), 16)
				let endB = parseInt(endColor.substring(4,6), 16)
				let b = Math.round((endB - startB)*value/100 + startB).toString(16)
				if(b.length == 1) {
					b = "0"+b
				}
				return "#"+r+g+b
			}
		}
	}
</script>
<style lang="scss" scoped>
	.gradient-slider {
		position: relative;
		border-radius: 50%;
		
		.progress-bar {
			position: absolute;
			top: 0;
			left: 0;
			border-radius: inherit;
			transition: width 0.2s;
			z-index: 1;
		}

		.gradient-slider-gap {
			position: relative;
			border-radius: inherit;
			transition: width 0.2s;
			z-index: 2;

			.gradient-slider-wrap {
				position: absolute;
				top: 50%;
				right: 0;
				transform: translate3d(50%, -50%, 0);
			}
		}
	}
</style>