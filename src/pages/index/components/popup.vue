<template>
	<view @touchmove.stop.prevent>
		<view
			class="qt-popup-class qt-bottom-popup"
			:class="{ 'qt-popup-show': show }"
			:style="{ background: bgColor, height: height ? height + 'rpx' : 'auto' }"
		>
			<view v-if="title" class="qt-popup-title">{{ title }}</view>
			<slot></slot>
		</view>
		<view
			v-if="mask"
			class="qt-popup-mask"
			:class="[show ? 'qt-mask-show' : '']"
			@tap="handleClose"
		></view>
	</view>
</template>

<script setup>
const props = defineProps(['show', 'title', 'mask', 'bgColor', 'height'])

const emit = defineEmits(['update:show'])

const handleClose = () => {
	if (!props.show) return
	emit('update:show', false)
}
</script>

<style>
.qt-bottom-popup {
	width: 100%;
	position: fixed;
	left: 0;
	right: 0;
	bottom: 0;
	z-index: 99999;
	visibility: hidden;
	transform: translate3d(0, 100%, 0);
	transform-origin: center;
	transition: all 0.3s cubic-bezier(0.41, 0.84, 0.41, 1);
	min-height: 20rpx;
}

.qt-popup-show {
	transform: translate3d(0, 0, 0);
	visibility: visible;
}

.qt-popup-mask {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background: rgba(0, 0, 0, 0.6);
	z-index: 99996;
	transition: all 0.3s ease-in-out;
	opacity: 0;
	visibility: hidden;
}

.qt-mask-show {
	opacity: 1;
	visibility: visible;
}

.qt-popup-title {
	font-size: 40upx;
	font-weight: 600;
	color: #3f3f3f;
	text-align: center;
	margin-top: 32upx;
}
</style>
