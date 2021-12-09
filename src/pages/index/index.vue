<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{ title }}</text>
		</view>
		<QtSelect v-model="selectKey" :columns="columns">
			<template #default="{ label }">{{ label }}</template>
		</QtSelect>
	</view>
</template>

<script>
import Select from './components/select.vue'
import QtSelect from './components/qt-select.vue'
export default {
	data() {
		return {
			title: 'Hello',
			selectKey: '',
			columns: [
				{ text: 'hh', value: 1 },
				{ text: 'ff', value: 2 },
				{ text: 'aa', value: 3 },
				{ text: 'dd', value: 4 },
			],
		}
	},
	onLoad() {},
	methods: {},
	components: { Select, QtSelect },
}
</script>

<style>
.content {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}

.logo {
	height: 200rpx;
	width: 200rpx;
	margin-top: 200rpx;
	margin-left: auto;
	margin-right: auto;
	margin-bottom: 50rpx;
}

.text-area {
	display: flex;
	justify-content: center;
}

.title {
	font-size: 36rpx;
	color: #8f8f94;
}
</style>
