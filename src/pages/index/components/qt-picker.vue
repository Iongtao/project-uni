<template>
	<view class="qt-picker">
		<picker-view
			class="qt-picker-view"
			:value="valueIndex"
			@change="changeHandle"
		>
			<picker-view-column
				v-for="(column, index) in computedColumns"
				:key="index"
			>
				<view
					v-for="columnItem in column"
					:key="columnItem.value"
					class="column-item"
				>
					{{ columnItem[columnsFieldNames['text']] }}
				</view>
			</picker-view-column>
		</picker-view>
		<view class="qt-picker-footer">
			<button class="qt-picker-footer-btn cancel" radius plain @tap="cancel">
				取消
			</button>
			<button class="qt-picker-footer-btn confirm" radius @tap="confirm">
				确定
			</button>
		</view>
	</view>
</template>

<script setup>
import { computed, ref, watch } from 'vue'

const isColumnOneFlat = (arr) => {
	return Array.isArray(arr) && !Array.isArray(arr[0])
}

const props = defineProps({
	columns: {
		type: Array,
		default: () => [],
	},
	columnsFieldNames: {
		type: Object,
		default: () => ({
			text: 'text',
			value: 'value',
			children: 'children',
		}),
	},
})

const emit = defineEmits(['cancel', 'confirm'])

// 索引值
const valueIndex = ref([])
// 索引所在列的值
const value = ref([])

const computedColumns = computed(() => {
	if (isColumnOneFlat(props.columns)) {
		let arr = []
		formatColumns(props.columns, 0, arr)
		return arr
	} else {
		return props.columns
	}
})

const formatColumns = (columns, columnIndex = 0, arr = []) => {
	columns.forEach((column, i) => {
		let index = valueIndex.value[columnIndex] || 0
		// 先把第index列的数据加到arr
		if (arr[columnIndex]) {
			arr[columnIndex].push(column)
		} else {
			arr[columnIndex] = [column]
		}

		if (index === i) {
			value.value[columnIndex] = column.value
			if (column.children && column.children.length) {
				formatColumns(column.children, columnIndex + 1, arr)
			}
		}
	})
}

// 设置默认位置
const setColumnIndex = (columnIndex, rowIndex) => {
	valueIndex.value[columnIndex] = rowIndex
}

const changeHandle = (e) => {
	valueIndex.value = e.detail.value
}

const confirm = () => {
	emit('confirm', value.value, valueIndex.value)
}
const cancel = () => {
	emit('cancel')
}
</script>

<style lang="scss" scoped>
.qt-picker {
	&-footer {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 36upx 24upx;
		box-shadow: 0 0 12upx 0 rgba($color: #000000, $alpha: 0.1);
		margin-top: 4upx;

		&-btn {
			flex: 1;
		}

		.cancel {
			margin-right: 32upx;
		}
	}

	&-view {
		height: 488upx;
	}

	.column-item {
		height: 68upx;
		line-height: 68upx;
	}
}
</style>
