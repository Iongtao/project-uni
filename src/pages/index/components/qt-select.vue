<template>
	<div
		:class="[
			'qt-select-content',
			!columns_info.length ? 'qt-select-placeholder' : '',
		]"
		@tap="showPicker = true"
	>
		<slot :label="label"></slot>
	</div>
	<Popup v-model:show="showPicker" mask :title="popupTitle">
		<QtPicker
			:ref="picker"
			:columns="columns_filter"
			:columns-field-names="columnFieldNames"
			@cancel="showPicker = false"
			@confirm="onConfirm"
		/>
	</Popup>
</template>

<script>
import { computed, ref, nextTick, watch } from 'vue'
import QtPicker from './qt-picker.vue'
import Popup from './popup.vue'

// 匹配v-model中的column信息
function setColumnInfoLoop(list, val, index, arr = []) {
	for (let i = 0; i < list.length; i++) {
		const item = list[i]
		const value = typeof val[index] === 'object' ? val[index].value : val[index]

		if (item.value === value) {
			arr[index] = { ...item, index: i }
			index++
			item.children && setColumnInfoLoop(item.children, val, index, arr)
			break
		}
	}
}

// 根据columnsNum列数裁剪列数
function cutColumnChildLoop(list = [], num, first, index = 0) {
	// 解决地址引用问题
	let _list = JSON.parse(JSON.stringify(list))
	// 根据索引给每列添加第一项
	if (first[index]) _list.unshift(first[index])

	let arr = []
	index++

	_list.forEach((item) => {
		if (index >= num) {
			arr.push({
				...item,
				children: undefined,
			})
		} else {
			arr.push({
				...item,
				children: cutColumnChildLoop(item.children, num, first, index),
			})
		}
	})

	return arr
}

export default {
	props: {
		modelValue: {
			type: [String, Number, Object, Array],
			default: () => [],
		},
		columns: {
			type: Array,
			default: () => [],
		},
		popupTitle: {
			type: String,
			default: '',
		},
		allInValue: {
			type: Boolean,
			default: false,
		},
		placeholder: {
			type: String,
			default: '请选择',
		},
		// 是否显示拼接每列的text
		isJointText: {
			type: Boolean,
			default: false,
		},
		columnFieldNames: {
			type: Object,
			default: () => ({
				text: 'text',
				values: 'values',
				children: 'children',
			}),
		},
		columnNum: {
			type: Number,
			default: 0, // 0代表无限制
		},
		// 每列第一项的内容
		// 格式：[{ label: '', value: ''}] index对应columns每列的索引
		// 该属性的length必需与columnNum一致 否则会出现多列或少列的情况。
		// 原因是van-picker默认根据第一项的children结构来显示列数
		columnFirst: {
			type: Array,
			default: () => [],
		},
	},
	emits: ['update:modelValue', 'confirm'],
	setup(props, { emit }) {
		const picker = ref(null)
		const showPicker = ref(false)

		// 根据columnNum裁剪到指定列 返回新的columns
		const columns_filter = computed(() => {
			let { columns, columnNum, columnFirst } = props
			// 如果columnNum列数没有的话 以columnFirst的长度为准
			columnNum = columnNum ? columnNum : columnFirst.length
			if (columnNum === 0) return columns
			const arr = cutColumnChildLoop(columns, columnNum, columnFirst)
			return arr
		})
		// 根据传入的值 获取对应的column项的内容
		const columns_info = computed(() => {
			let arr = []
			console.log('columns_filter', columns_filter.value)
			if (columns_filter.value.length) {
				const value =
					typeof props.modelValue === 'object' &&
					props.modelValue instanceof Array
						? props.modelValue
						: [props.modelValue]
				setColumnInfoLoop(columns_filter.value, value, 0, arr)
			}
			console.log('arr', arr)
			return arr
		})
		// 展示选择的信息
		const label = computed(() => {
			let label = ''
			// 未选择时候默认显示占位内容
			if (!columns_info.value.length) {
				// 默认拿第一项的最后一列的label信息
				if (props.columnFirst.length) {
					label =
						props.columnFirst[props.columnFirst.length - 1][
							props.columnFieldNames['text']
						]
				} else if (props.placeholder) label = props.placeholder
			} else {
				if (props.isJointText) {
					// 拼接每列的文本
					label = columns_info.value.reduce(
						(pre, cur, idx, arr) =>
							(pre +=
								cur[props.columnFieldNames['text']] +
								(idx === arr.length - 1 ? '' : '-')),
						''
					)
				} else {
					// 如果是第一项 则取该项的最后一列的label信息
					if (columns_info.value[0].index === 0) {
						label =
							columns_info.value[columns_info.value.length - 1][
								props.columnFieldNames['text']
							]
					} else {
						// 默认根据最后一列的value值，读取其文本内容。没有依次往前取
						for (let i = columns_info.value.length - 1; i >= 0; i--) {
							if (columns_info.value[i].value) {
								label = columns_info.value[i][props.columnFieldNames['text']]
								break
							}
						}
					}
				}
				// 都没有就取第一个的文本内容
				if (label === '')
					label = columns_info.value[0][props.columnFieldNames['text']]
			}

			console.log('label', label)
			return label
		})
		// 每次打开根据value设置指定列的位置
		watch(showPicker, (n) => {
			console.log('showPicker', n)
			if (n) {
				nextTick(() => {
					picker.value &&
						columns_info.value.forEach((v, i) => {
							picker.value.setColumnIndex(i, v.index)
						})
				})
			}
		})
		// 点击确认
		const onConfirm = (arr = []) => {
			let val = arr
			if (
				!(
					typeof props.modelValue === 'object' &&
					props.modelValue instanceof Array
				)
			) {
				val = val[0]
			}
			emit('update:modelValue', val)
			emit('confirm', val)
			showPicker.value = false
		}
		return {
			showPicker,
			picker,
			label,
			columns_info,
			columns_filter,
			onConfirm,
		}
	},
	components: { QtPicker, Popup },
}
</script>

<style lang="scss" scoped>
.qt-select-placeholder {
	color: #c5c5c5;
}
</style>
