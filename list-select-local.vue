<template>
	<view
		class="list-select"
		:style="{
			'--bg-color': bgColor,
		}"
	>
		<!-- 蒙版 -->
		<view v-if="_show_out_mask" class="out-mask" @tap="closeAllPopup()"></view>

		<view class="header">
			<!-- 搜索 -->
			<view class="search">
				<view class="search-input" @tap="!search_on ? searchInputBlur() : ''">
					<view class="search-before" v-if="!search_on" @tap="searchInputBlur()">
						<view class="iconfont icon-search"></view>
						<view class="color-ccc ml-10">{{ search_value || searchPlaceHolder }}</view>
					</view>

					<block v-else>
						<view class="iconfont icon-search"></view>
						<input focus="true" @blur="searchInputBlur()" class="flex1 fs-28 ml-20" type="text" v-model="search_value" />
					</block>
				</view>
			</view>

			<!-- 筛选 -->
			<view v-if="!is_searched && !search_on && (_filter_common.length || filterSort.length > 1)" class="filter">
				<view class="filter-box">
					<view
						v-for="(item, index) in _filter_common"
						:key="index"
						@tap="filterShowMenu(index)"
						:class="item.selected_label ? 'on' : ''"
					>
						<view>{{ item.selected_label || item.title }}</view>
						<view class="iconfont icon-arrowdown " :class="actived_index == index ? 'icon-arrowup' : 'icon-arrowdown'"></view>
					</view>

					<view v-if="filterSort.length > 1" @tap="sortShowMenu()" :class="sort_index != -1 ? 'on' : ''">
						<view>{{ filterSort[sort_index].title || '排序' }}</view>
						<view class="iconfont icon-paixu" style="font-size: 28rpx;"></view>
					</view>
				</view>

				<view class="filter-content" v-if="filter_popup || sort_popup">
					<view v-if="sort_popup" class="filter-content-list">
						<view
							v-for="(option, index) in filterSort"
							:key="index"
							:class="sort_index == index ? 'filter-content-list-item-active' : 'filter-content-list-item-default'"
							@tap="selectSortType(index)"
						>
							<text>{{ option.title }}</text>
							<view class="flex1"></view>
							<view v-if="sort_index == index" class="iconfont icon-gou1"></view>
						</view>
					</view>

					<block v-else>
						<view v-if="_filter_common[actived_index].style == 'line'" class="filter-content-list">
							<view
								v-for="(option, index) in _filter_common[actived_index].options"
								:key="index"
								:class="option.is_selected ? 'filter-content-list-item-active' : 'filter-content-list-item-default'"
								@tap="filterSelectSingleOption(index, option.value)"
							>
								<text>{{ option.label }}</text>
								<view class="flex1"></view>
								<view v-if="option.is_selected" class="iconfont icon-gou1"></view>
							</view>
						</view>

						<view v-if="_filter_common[actived_index].style == 'block'" class="filter-content-block">
							<view
								v-for="(option, index) in _filter_common[actived_index].options"
								:key="index"
								:class="option.is_selected ? 'filter-content-block-item-active' : 'filter-content-block-item-default'"
								@tap="filterSelectSingleOption(index, option.value)"
							>
								<view class="flex1">{{ option.label }}</view>
							</view>
						</view>
					</block>
				</view>
			</view>
		</view>
		<view :style="'height:' + header_height + 'px'"></view>

		<!-- 搜索历史 -->
		<view v-if="search_history.length && (is_searched || search_on)" class="search-history">
			<view class="search-history-header">
				<view class="search-history-title">搜索历史</view>
				<view class="flex1"></view>
				<view @tap="clearSearchHistory" class="iconfont icon-iconfont-shanchu"></view>
			</view>
			<view class="search-history-box" v-for="(item, index) in search_history" :key="index" @tap="chooseHistroy(item)">
				{{ item }}
			</view>
		</view>

		<view class="search-after-border"></view>

		<!-- 搜索后列表 -->
		<view v-if="is_searched || search_on" class="list">
			<block v-if="search_result.length">
				<!-- 搜索后列表 -->
				<view v-for="(item, index) in search_result" :key="index" class="list-item" @tap="selectOne(item, index, 'search_result')">
					<slot :item="item" :item_type="itemType">
						<block v-if="itemType == 'member'"><member :member="item"></member></block>
						<block v-else-if="itemType == 'staff'">
							<staff :full="false" :selected="has_selected ? has_selected.has(item.id) : null" :staff="item"></staff>
						</block>
					</slot>
				</view>
			</block>
			<no-data :bgColor="bgColor" v-if="is_searched && !search_result.length" tip="什么也没有搜到~"></no-data>
		</view>

		<!-- 列表 -->
		<view v-if="!is_searched && !search_on" class="list">
			<block v-if="_list.length">
				<block v-for="(item, index) in _list" :key="index">
					<view v-if="_show_pinyin_index && pinyin_index_point_name[index]" class="list-index-header" :id="'index-' + index">
						{{ pinyin_index_point_name[index] }}
					</view>
					<view class="list-item" @tap="selectOne(item, index, '_list')">
						<slot v-if="useSlot" :item="item" :item_type="itemType"></slot>
						<block v-else>
							<block v-if="itemType == 'member'"><member :member="item"></member></block>
							<block v-else-if="itemType == 'staff'">
								<staff :full="false" :selected="has_selected ? has_selected.has(item.id) : null" :staff="item"></staff>
							</block>
						</block>
					</view>
				</block>
			</block>
			<no-data :bgColor="bgColor" v-else :tip="filter_conditions.length ? '换个条件试试?' : listNone.tip"></no-data>
		</view>

		<!-- 索引 -->
		<view v-if="_show_pinyin_index" class="index">
			<view v-for="(name, index) in pinyin_index_point_name" :key="index" @click="selectIndex(name)">{{ name }}</view>
		</view>

		<view v-if="hasSelected || isMutiple" class="confirm_btn" @tap="confirmMutipleSelect">确认</view>
	</view>
</template>

<script>
import Member from '@/components/core/member/member.vue';
import Staff from '@/components/core/staff/staff.vue';

/**
 * 列表筛选 本地筛选 支持拼音索引 适应于数量不多字段固定的列表
 */
export default {
	components: {
		Member,
		Staff,
	},
	name: 'ListSelectLocal',
	props: {
		list: {
			//列表数据
			type: Array,
			default: [],
		},
		listNone: {
			//列表空显示
			type: Object,
			default: {
				tip: '空空如也~',
			},
		},
		itemType: {
			//item类型 对应组件item-后缀
			type: String,
			required: true,
		},
		searchBy: {
			// 搜索根据什么字段
			type: Array,
			required: true,
		},
		searchPlaceHolder: {
			//搜索栏占位符
			type: String,
			required: true,
		},
		filterCommon: {
			// 普通单选筛选
			type: Array,
			default: [],
		},
		usePinyin: {
			//使用拼音 字符排序会通过拼音排序
			type: Boolean && Object,
			default: {
				use: true,
				new_field: '_py', //拼音的字段
			},
		},
		usePinyinIndex: {
			//使用拼音索引
			type: Boolean,
			default: true,
		},
		useSlot: {
			//使用slot
			type: Boolean,
			default: true,
		},
		filterSort: {
			// 排序筛选
			type: Array,
			default: [
				{
					title: '姓名排序',
					field: 'name', //通过某个字段排序
					by: 'asc', //desc
					type: 'string', //字符排序 number//数字排序
					is_default: true,
				},
			],
		},
		isMutiple: {
			//多选
			type: Boolean,
			default: false,
		},
		mutipleLimit: {
			//多选数量限制
			type: Object,
			default: {
				max: null,
				min: null,
			},
		},
		hasSelected: {
			//已选择的
			type: [Object, Array],
			default: null,
		},
		bgColor: {
			type: String,
			default: '#f6f6f6',
		},
	},
	data() {
		return {
			header_height: 0,

			pin_yin_sort_list: [], //持久化存储拼音排序后的数据

			pinyin_index_point_name: {}, //拼音索引 坐标=>名称 10:D
			pinyin_index_name_point: {}, //拼音索引 名称=>坐标 D:10

			//搜索
			search_on: false,
			search_value: '',

			search_timer: {},
			is_searched: false,

			search_history: [],
			search_result: [],

			//筛选
			actived_index: -1, //目前打开的
			selected_indexs: {}, //被选择的
			filter_popup: false,
			filter_conditions: [],

			// 排序
			sort_popup: false,
			sort_index: -1,

			need_mark_selected: false, //是否需要标记已选
			has_selected: new Set(), //已经选择的 id
		};
	},
	computed: {
		/**
		 * 筛选后列表
		 */
		_list: {
			get: function() {
				this.$base.showLoading();
				let new_list = [];

				if (this.filter_conditions.length) {
					new_list = this._list_default.filter(item => {
						return this.filter_conditions.every(condition => {
							return item[condition.field] == condition.value;
						});
					});

					let sort_type = this.getSortTypeInUse();
					if (sort_type.type === 'string' && this.usePinyin.use) {
						//筛选后重建索引
						this.buildPinYinIndex(new_list);
					}
				} else {
					new_list = this._list_default;
				}

				uni.hideLoading();
				return new_list;
			},
			set: function() {},
		},

		/**
		 * 担任拼音的字段
		 */
		_pinyin_new_field: function() {
			return this.usePinyin.new_field || '_py';
		},

		/**
		 * 初始列表 / 排序后列表
		 */
		_list_default: function() {
			if (!this.list.length) {
				return [];
			}

			if (!this.need_mark_selected) {
				if (this.isMutiple) {
					this.need_mark_selected = true;
					if (this.hasSelected && this.hasSelected instanceof Array) {
						//多选 且提供已选 标记已选
						this.hasSelected.forEach(item => {
							this.has_selected.add(item.id);
						});
					}
				}
				if (!this.isMutiple && this.hasSelected instanceof Object) {
					//单选 且提供已选 标记已选
					this.list.some(item => {
						if (this.hasSelected.id == item.id) {
							this.has_selected.add(item.id);
							this.need_mark_selected = true;
							return true;
						}
					});
				}
			}

			let sort_type = this.getSortTypeInUse();

			if (sort_type.type === 'string' && this.usePinyin.use) {
				if (!this.pin_yin_sort_list.length) {
					let pinyin_processor = require('pinyin');
					let pin_yin_list = this.list.map(item => {
						let pinyin = {};
						pinyin[this._pinyin_new_field] = pinyin_processor(item[sort_type.field], {
							style: pinyin_processor.STYLE_FIRST_LETTER,
						})[0][0]
							.substr(0, 1)
							.toUpperCase();
						return Object.assign(item, pinyin);
					});

					this.pin_yin_sort_list = pin_yin_list.sort((a, b) => {
						return a[this._pinyin_new_field].localeCompare(b[this._pinyin_new_field]);
					});
					this.buildPinYinIndex(this.pin_yin_sort_list);
				}
				return this.pin_yin_sort_list;
			} else {
				return this.list.sort((a, b) => {
					let compare = a[sort_type.field] - b[sort_type.field];
					return sort_type.by === 'asc' ? compare : 0 - compare;
				});
			}
		},

		/**
		 * 普通筛选
		 */
		_filter_common: function() {
			if (!this.filterCommon.length) {
				return [];
			}

			return this.filterCommon.map(filter => {
				if (!filter.hasOwnProperty('options_config')) {
					filter.options_config = {};
				}
				filter.options_config = Object.assign(
					{
						unshift_all: true, //options自动向前插入 全部选项 (默认)
						default: null,
						output: null,
					},
					filter.options_config,
				);

				filter.options = filter.options.map(option => {
					return Object.assign(option, {
						is_selected: false,
					});
				});

				if (filter.options_config.unshift_all) {
					filter.options.unshift({
						label: '全部',
						value: null,
						is_selected: filter.options_config.default === null,
					});
				}

				return Object.assign(filter, {
					is_selected: false,
					selected_label: '',
				});
			});
		},

		/**
		 * 缓存key
		 */
		_search_history_key: function() {
			let auth = this.$cache.get('auth');
			let shop = this.$cache.get('shop');
			let key = `${this.itemType}_search_history_${auth.id}_${shop.id}`;
			this.search_history = this.$cache.get(key) || [];
			return key;
		},

		/**
		 * 检测搜索字段是否合法
		 */
		_search_field_valid: function() {
			if (!this.list.length || !this.searchBy.length) {
				return true;
			}
			return this.searchBy.every(by => {
				return this.list[0].hasOwnProperty(by);
			});
		},

		_show_out_mask: function() {
			return this.filter_popup || this.sort_popup;
		},

		_show_pinyin_index: function() {
			return (
				!this._show_out_mask &&
				!this.is_searched &&
				!this.search_on &&
				this.sort_index != -1 &&
				this.filterSort[this.sort_index].type === 'string' &&
				this.usePinyin.use &&
				this.usePinyinIndex
			);
		},
	},
	onUnload() {
		if (this.search_timer) {
			clearInterval(this.search_timer);
			this.search_timer = null;
		}
	},
	mounted() {
		uni.createSelectorQuery()
			.in(this)
			.select('.header')
			.boundingClientRect(rect => {
				this.header_height = rect.height;
			})
			.exec();
	},
	methods: {
		selectOne(item, index, key) {
			if (this.need_mark_selected) {
				if (!this.isMutiple) {
					this[key].forEach((item, i) => {
						if (index == i) {
							if (this.has_selected.has(item.id)) {
								this.has_selected.delete(item.id);
							} else {
								this.has_selected.add(item.id);
							}
						} else {
							if (this.has_selected.has(item.id)) {
								this.has_selected.delete(item.id);
							}
						}
					});
					this.$forceUpdate();
				} else {
					let tail = !this.has_selected.has(this[key][index].id);
					let check_result = this.checkMutipleLimit(this[key], tail);
					if (!check_result) {
						return;
					}
					if (tail) {
						this.has_selected.add(this[key][index].id);
					} else {
						this.has_selected.delete(this[key][index].id);
					}
					this.$forceUpdate();
				}
			} else {
				this.confirmSelect(item);
			}
		},

		confirmMutipleSelect() {
			this.confirmSelect(this._list.filter(item => this.has_selected.has(item.id)));
		},

		confirmSelect(item) {
			this.$emit('change', item);
		},

		/**
		 * 检测多选数量限制
		 * @param {Array} arr 检测目标数组
		 * @param {Boolean} operator true+  false-
		 */
		checkMutipleLimit(arr, operator) {
			let count = 0;
			let result = true;
			let trig_key = '';
			for (let key in this.mutipleLimit) {
				if (this.mutipleLimit[key] !== null) {
					if (key !== 'max' && key !== 'min') {
						continue;
					}
					count = this.has_selected.size + (operator ? +1 : -1);
					if (key === 'max') {
						result = count <= this.mutipleLimit[key];
					} else if (key === 'min') {
						result = count >= this.mutipleLimit[key];
					}
					if (!result) {
						trig_key = key;
						break;
					}
				}
			}
			if (!result) {
				this.$emit('mutipleLimit', {
					key: trig_key,
					limit: this.mutipleLimit[trig_key],
					count,
				});
				return false;
			}
			return result;
		},

		//单选选中
		filterSelectSingleOption(index, value) {
			this.filter_conditions = this.filter_conditions.filter(condition => condition.filter_type_index != this.actived_index);

			if (!this._filter_common[this.actived_index].options[index].is_selected) {
				this._filter_common[this.actived_index].options = this._filter_common[this.actived_index].options.map(option => {
					option.is_selected = false;
					return option;
				});
				this._filter_common[this.actived_index].options[index].is_selected = true;
				this._filter_common[this.actived_index].is_selected = true;

				if (this._filter_common[this.actived_index].options[index].value === null) {
					//选择全部时 显示title
					this._filter_common[this.actived_index].selected_label = '';
				} else {
					this._filter_common[this.actived_index].selected_label = this._filter_common[this.actived_index].options[index].label;

					if (this._filter_common[this.actived_index].options_config.output) {
						let condition = {};
						condition[this._filter_common[this.actived_index].field] = this._filter_common[this.actived_index].options[
							index
						].value;
						this.$emit('filter', {
							value: condition,
						});
					} else {
						this.filter_conditions.push({
							field: this._filter_common[this.actived_index].field,
							value: this._filter_common[this.actived_index].options[index].value,
							filter_type_index: this.actived_index,
						});
					}
				}
			} else {
				this._filter_common[this.actived_index].options[index].is_selected = false;
				this._filter_common[this.actived_index].is_selected = false;
				this._filter_common[this.actived_index].selected_label = '';
			}
			this.filterPopupClose();
		},

		/**
		 * 建立拼音索引
		 */
		buildPinYinIndex(pin_yin_sort_list) {
			this.pinyin_index_name_point = {};
			this.pinyin_index_point_name = {};
			pin_yin_sort_list.forEach((item, index) => {
				if (!this.pinyin_index_name_point.hasOwnProperty(item[this._pinyin_new_field])) {
					this.pinyin_index_name_point[item[this._pinyin_new_field]] = index;
					this.pinyin_index_point_name[index] = item[this._pinyin_new_field];
				}
			});
		},

		/**
		 * 获取当前使用的排序方式
		 */
		getSortTypeInUse() {
			let sort_type = {};
			if (this.sort_index == -1) {
				sort_type = this.filterSort.find((sort, index) => {
					if (sort.is_default) {
						this.sort_index = index;
						return true;
					}
				});
			} else {
				sort_type = this.filterSort[this.sort_index];
			}
			return sort_type;
		},

		getPageOffset() {
			return new Promise(resolve => {
				uni.createSelectorQuery()
					.selectViewport()
					.scrollOffset(e => {
						resolve(e.scrollTop);
					})
					.exec();
			});
		},

		async selectIndex(name) {
			this.$base.showToast(name);
			let id = `#index-${this.pinyin_index_name_point[name]}`;
			let page_offset_top = await this.getPageOffset();

			uni.createSelectorQuery()
				.in(this)
				.select(id)
				.boundingClientRect(index_data => {
					if (index_data.top) {
						uni.pageScrollTo({
							duration: 600, //过渡时间
							scrollTop: page_offset_top + index_data.top - this.header_height, //到达距离顶部的top值
						});
					}
				})
				.exec();
		},

		search() {
			clearInterval(this.search_timer);
			this.search_timer = null;

			this.search_result = [];

			this.$base.showLoading('搜索中');
			this.list.forEach(item => {
				this.searchBy.forEach(by => {
					if (item[by].includes(this.search_value)) {
						this.search_result.push(item);
					}
				});
			});
			this.recordSearchHistory();
			this.is_searched = true;
			uni.hideLoading();
		},

		recordSearchHistory() {
			if (!this.search_value) {
				return;
			}
			let now_search_history = this.search_history;
			//是否已存在
			let is_exsits = now_search_history.findIndex(item => item === this.search_value);
			if (is_exsits !== -1) {
				now_search_history.splice(is_exsits, 1);
			}
			now_search_history.unshift(this.search_value);
			if (now_search_history.length > 10) {
				now_search_history.pop();
			}
			this.$cache.set(this._search_history_key, now_search_history, 60 * 60 * 24 * 7);
		},

		chooseHistroy(item) {
			this.search_value = item;
		},

		clearSearchHistory() {
			this.$cache.remove(this._search_history_key);
			this.search_history = [];
		},

		searchInputBlur(e) {
			this.filterPopupClose();
			this.search_on = !this.search_on;
			if (this.search_on) {
				uni.pageScrollTo({
					duration: 300, //过渡时间
					scrollTop: 0, //到达距离顶部的top值
				});
			}
		},

		selectSortType(index) {
			if (this.sort_index != index) {
				this.sort_index = index;
			}

			this.sortPopupClose();
		},

		sortShowMenu() {
			this.sort_just_reverse = false;
			this.sort_popup = !this.sort_popup;
		},

		filterShowMenu(index) {
			if (this.actived_index === index) {
				this.filterPopupClose();
				return;
			}
			this.actived_index = index;
			this.filterPopupOpen();
		},

		sortPopupClose() {
			this.sort_popup = false;
		},

		sortPopupOpen() {
			this.sort_popup = true;
		},

		filterPopupClose() {
			this.actived_index = -1;
			this.filter_popup = false;
		},

		closeAllPopup() {
			this.filter_popup = this.sort_popup = false;
		},

		filterPopupOpen() {
			this.filter_popup = true;
		},
	},
	watch: {
		//延时搜索
		search_value(value) {
			if (!this._search_field_valid) {
				clearInterval(this.search_timer);
				this.search_on = false;
				console.error('搜索字段不存在或者错误!');
				return;
			}

			let i = 1;
			let j = 1;
			if (this.search_timer) {
				clearInterval(this.search_timer);
			}
			this.search_timer = setInterval(() => {
				if (this.search_value.trim()) {
					++i;
					if (i == 4) {
						this.search();
					}
				} else {
					++j;
					if (j == 4) {
						this.is_searched = false;
						this.search_on = false;
						clearInterval(this.search_timer);
					}
				}
			}, 200);
		},
	},
};
</script>

<style lang="scss">
@import './style.scss';
page {
	border-top: inherit !important;
}

.list-select {
	background-color: var(--bg-color);
}

.list-select .list {
	padding-bottom: 200rpx;
}

.confirm_btn {
	width: 690rpx;
	height: 74rpx;
	border-radius: 20rpx;
	padding: 0;
	font-size: 26rpx;
	color: #fff;
	line-height: 74rpx;
	text-align: center;
	background-color: $color-light;
	position: fixed;
	bottom: 30rpx;
	left: 30rpx;
}
</style>
