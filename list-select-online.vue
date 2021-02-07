<template>
	<view class="list-select">
		<!-- 蒙版 -->
		<view v-if="_show_out_mask" class="out-mask" @tap="commonFilterPopupClose()"></view>

		<view class="header">
			<!-- 搜索 -->
			<view class="search" v-if="useSearch">
				<view class="search-input" @tap="!search_on ? searchInputBlur() : ''">
					<view class="search-before" v-if="!search_on" @tap.stop="searchInputBlur()">
						<view class="iconfont icon-search"></view>
						<view class="color-ccc ml-10">{{ search_value || searchPlaceHolder }}</view>
					</view>

					<block v-else>
						<view class="iconfont icon-search"></view>
						<input
							:focus="search_input_focus"
							@blur="searchInputBlur()"
							class="flex1 fs-28 ml-20"
							type="text"
							v-model="search_value"
						/>
					</block>
				</view>
			</view>

			<!-- 筛选 -->
			<view class="filter" v-if="_filters.length">
				<view class="filter-box">
					<block v-for="(item, index) in _filters" :key="index">
						<block v-if="item.style === 'slot'">
							<slot name="slot_filter"></slot>
						</block>
						<view
							v-else-if="!item.show || (item.show.result === true && !item.show.hidden)"
							@tap="commonFilterShowMenu(index)"
							:class="item.selected_label ? 'on' : ''"
						>
							<view>{{ item.selected_label || item.title }}</view>
							<view
								class="iconfont icon-arrowdown "
								:class="actived_index == index ? 'icon-arrowup' : 'icon-arrowdown'"
							></view>
						</view>
					</block>
				</view>

				<view v-if="common_filter_popup" class="filter-content">
					<view v-if="_filters[actived_index].style === 'line'" class="filter-content-list">
						<block v-for="(option, index) in _filters[actived_index].options" :key="index">
							<view
								v-if="!option.type || option.type === 'common'"
								:class="option.is_selected ? 'filter-content-list-item-active' : 'filter-content-list-item-default'"
								@tap="filterSelectSingleOption(index)"
							>
								<text>{{ option[_filters[actived_index].options_config.label_field] }}</text>
								<view class="flex1"></view>
								<view v-if="option.is_selected" class="iconfont icon-gou1"></view>
							</view>

							<block v-if="option.type === 'component'">
								<block v-if="option.component == 'pyh-rdtpicker'">
									<view
										:class="option.is_selected ? 'filter-content-list-item-active' : 'filter-content-list-item-default'"
										@click="dateArrShow(index, option[_filters[actived_index].options_config.value_field])"
									>
										{{ date_arr_selected_str || option[_filters[actived_index].options_config.label_field] }}
									</view>
									<rangeDatePick
										:show="date_arr_is_show"
										@showchange="dateArrShow(index, option[_filters[actived_index].options_config.value_field])"
										start="2010-01-01"
										end="2200-01-01"
										:value="date_arr_selected"
										@change="dateArrBindChange()"
										themeColor="#3e8ef7"
										fields="day"
									></rangeDatePick>
								</block>
							</block>
						</block>
					</view>

					<view v-else-if="_filters[actived_index].style === 'block'" class="filter-content-block">
						<view
							v-for="(option, index) in _filters[actived_index].options"
							:key="index"
							:class="option.is_selected ? 'filter-content-block-item-active' : 'filter-content-block-item-default'"
							@tap="filterSelectSingleOption(index)"
						>
							<view class="flex1">{{ option[_filters[actived_index].options_config.label_field] }}</view>
						</view>
					</view>

					<view v-else-if="_filters[actived_index].style === 'multiplex'" class="filter-content-multiplex">
						<view v-for="(filter, filter_index) in _filters[actived_index].children" :key="filter_index">
							<block v-if="!filter.show || filter.show.result === true">
								<view class="filter-content-multiplex-title">{{ filter.title }}</view>
								<view class="filter-content-block">
									<view
										v-for="(option, option_index) in filter.options"
										:key="option_index"
										:class="
											option.is_selected ? 'filter-content-block-item-active' : 'filter-content-block-item-default'
										"
										@tap="filterSelectSingleOption(option_index, filter_index)"
									>
										<view>{{ option[filter.options_config.label_field] }}</view>
									</view>
								</view>
							</block>
						</view>
					</view>
				</view>
			</view>

			<view v-if="useSearch || filters.length" class="search-after-border"></view>
		</view>
		<view :style="'height:' + header_height + 'px'"></view>
		<!-- 搜索历史 -->
		<view v-if="search_history.length && search_on" class="search-history">
			<view class="search-history-header">
				<view class="search-history-title">搜索历史</view>
				<view class="flex1"></view>
				<view @tap="clearSearchHistory" class="iconfont icon-iconfont-shanchu"></view>
			</view>
			<view class="search-history-box" v-for="(item, index) in search_history" :key="index" @tap="chooseHistroy(item)">
				{{ item }}
			</view>
		</view>

		<!-- 列表 -->
		<view class="list"><slot></slot></view>
	</view>
</template>

<script>
import { getNowFormatDate } from '@/common/helper/time.js';

import rangeDatePick from '@/components/pyh-rdtpicker/pyh-rdtpicker.vue';

/**
 * 列表筛选 实时请求接口
 */
export default {
	name: 'ListSelectOnline',
	components: {
		rangeDatePick,
	},
	props: {
		filters: {
			// 筛选
			type: Array,
			default: [],
		},
		useSearch: {
			type: Boolean,
			default: true,
		},
		searchPlaceHolder: {
			//搜索栏占位符
			type: String,
			required: true,
		},
		listNone: {
			//列表空显示
			type: Object,
			default: {
				tip: '空空如也~',
			},
		},
		pageKey: {
			//页面key标识 不设置则无法存储页面搜索记录
			type: String,
			default: '',
		},
	},
	data() {
		return {
			header_height: 0,

			//搜索
			search_on: false,
			search_value: '',
			search_timer: {},

			search_history: [],

			//筛选
			actived_index: -1, //目前打开的
			selected_indexs: {}, //被选择的

			common_filter_popup: false,
			filter_common_cached: false, //存储已经初始化完成的筛选框
			has_condition_show: false, //是否有跟进条件显示隐藏的筛选

			conditions_selected: {}, //已经选择的所有筛选条件

			//日期区间
			date_arr_is_show: false,
			date_arr_selected: [],
			date_arr_selected_str: '',

			search_input_focus: false, //输入框是否聚焦
		};
	},
	computed: {
		/**
		 * 普通筛选
		 */
		_filters: {
			get: function() {
				if (!this.filters.length) {
					this.search_input_focus = true;
					this.search_on = true;
					return [];
				}
				if (this.filter_common_cached !== false) {
					return this.filter_common_cached;
				}

				let initFilter = filters => {
					return filters.map(filter => {
						if (filter.style === 'slot') {
							return filter
						}
						if (filter.style === 'multiplex') {
							filter.children = initFilter(filter.children);
							return filter;
						}
						return this.rebuildFilter(filter);
					});
				};

				this.filter_common_cached = initFilter(this.filters);

				this.$emit('filter', {
					value: this.conditions_selected,
				});

				return this.filter_common_cached;
			},
			set: function() {},
		},

		/**
		 * 缓存key
		 */
		_search_history_key: function() {
			if (!this.pageKey) {
				return '';
			}
			let auth = this.$cache.get('auth');
			let shop = this.$cache.get('shop');
			let key = `${this.pageKey}_search_history_${auth.id}_${shop.id}`;
			this.search_history = this.$cache.get(key) || [];
			return key;
		},

		_show_out_mask: function() {
			return this.common_filter_popup;
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

		this.date_arr_selected = [getNowFormatDate()];
	},
	methods: {
		/**
		 * 初始化filter
		 * 1.初始化默认选项
		 * 2.追加`全部`选项
		 * 3.选中默认
		 * 4.检测动态筛选是否显示
		 * @param {Object} filter
		 */
		rebuildFilter(filter) {
			if (!filter.hasOwnProperty('options_config')) {
				filter.options_config = {};
			}
			if (!filter.hasOwnProperty('show')) {
				filter.show = {};
			}
			filter.options_config = Object.assign(
				{
					type: 'static', //类型 static(默认) / dynamic
					function: '', //dynamic 时必填
					cache: true,
					unshift_all: true, //options自动向前插入 全部选项 (默认)
					label_field: 'label',
					value_field: 'value',
					default: null,
				},
				filter.options_config,
			);
			filter.show = Object.assign(
				{
					type: 'all', //except / only / all
					result: true,
					hidden: false,
					condition_field: '',
					condition_values: [],
				},
				filter.show,
			);

			let filter_selected_label = '';
			let filter_is_selected = false;

			if (filter.options.length) {
				filter.options = filter.options.map((option, index) => {
					let is_selected = false;
					if (filter.options_config.default !== null && +filter.options_config.default === index) {
						is_selected = true;
						filter_is_selected = true;

						if (option.value !== null && this.conditions_selected[filter.field] !== option.value) {
							filter_selected_label = option[filter.options_config.label_field];
							this.conditions_selected[filter.field] = option[filter.options_config.value_field];
						}
					}
					return Object.assign(option, {
						is_selected,
					});
				});
			}

			if (filter.options_config.unshift_all) {
				filter.options.unshift({
					[filter.options_config.label_field]: '全部',
					[filter.options_config.value_field]: null,
					is_selected: filter.options_config.default === null,
				});
			}

			if (!filter.show.hidden && filter.show.type !== 'all') {
				this.has_condition_show = true;
				filter = this.checkDynamicFilterExistCondition(filter);
			}

			return Object.assign(filter, {
				is_selected: filter_is_selected,
				selected_label: filter_selected_label,
			});
		},

		/**
		 * 加载需要动态获取参数的选项
		 * @param {Object} filter
		 */
		async initDynamicFilterOptions(filter) {
			if (filter.options_config.type === 'dynamic' && filter.show.result) {
				if (!filter.options_config.request_times || !filter.options_config.cache) {
					let options = await this.$parent[filter.options_config.function]();

					if (filter.options_config.unshift_all) {
						filter.options = [filter.options[0]].concat(options);
					} else {
						filter.options = options;
					}
					if (filter.options_config.hasOwnProperty('request_times')) {
						filter.options_config.request_times++;
					} else {
						filter.options_config = Object.assign(filter.options_config, {
							request_times: 1,
						});
					}
				}
			}
			return Promise.resolve(filter);
		},

		/**
		 * 检查动态显示的筛选是否显示 (全部)
		 */
		checkDynamicFiltersExistCondition(filters) {
			return filters.map(filter => {
				if (filter.style === 'multiplex') {
					filter.children = this.checkDynamicFiltersExistCondition(filter.children);
					return filter;
				}
				return this.checkDynamicFilterExistCondition(filter);
			});
		},

		/**
		 * 检查动态显示的筛选是否显示 (单)
		 */
		checkDynamicFilterExistCondition(filter) {
			if (!filter.show) {
				return filter;
			}

			if (filter.show.type === 'all') {
				filter.show.result = true;
				return filter;
			}

			let assert = filter.show.condition_values.includes(this.conditions_selected[filter.show.condition_field]);
			filter.show.result = filter.show.type === 'only' ? assert : !assert;

			if (!filter.show.result && filter.selected_label) {
				//清除已经选择的 并将其隐藏
				filter.options = filter.options.map(option => {
					option.is_selected = option[filter.options_config.value_field] === null;
					return option;
				});
				filter.selected_label = '';
				delete this.conditions_selected[filter.field];
			}

			return filter;
		},

		/**
		 * 打开筛选框
		 * @param {Object} index
		 */
		async commonFilterShowMenu(index) {
			if (this.actived_index === index) {
				this.commonFilterPopupClose();
				return;
			}
			let filter = this._filters[index];
			if (filter.style === 'multiplex') {
				filter.children = filter.children.map((children_filter, index) => {
					(async () => {
						children_filter = await this.initDynamicFilterOptions(children_filter);
					})();
					return children_filter;
				});
			} else {
				filter = await this.initDynamicFilterOptions(this._filters[index]);
			}
			this._filters[index] = filter;
			this.actived_index = index;
			this.filterPopupOpen();
		},

		/**
		 * 单选选中
		 *@param {Number}  option_index options的下标
		 *@param {Number}  value 值
		 *@param {Number}  multiplex_filter_index 复合选择下的filter的下标
		 */
		filterSelectSingleOption(option_index, multiplex_filter_index = null) {
			let filter = {};
			if (multiplex_filter_index !== null) {
				filter = this._filters[this.actived_index].children[multiplex_filter_index];
			} else {
				filter = this._filters[this.actived_index];
			}
			let options = filter.options;
			let option_clicked = options[option_index];

			if (!option_clicked.is_selected) {
				options = options.map(option => {
					option.is_selected = false;
					return option;
				});

				option_clicked.is_selected = true;
				filter.is_selected = true;

				if (option_clicked[filter.options_config.value_field] === null) {
					//选择全部时 显示title
					filter.selected_label = '';
					delete this.conditions_selected[filter.field];
				} else {
					filter.selected_label = option_clicked[filter.options_config.label_field];

					this.conditions_selected[filter.field] = option_clicked[filter.options_config.value_field];
				}
			} else {
				option_clicked.is_selected = false;
				filter.is_selected = false;
				filter.selected_label = '';
				delete this.conditions_selected[filter.field];
			}

			if (this.has_condition_show) {
				this._filters = this.checkDynamicFiltersExistCondition(this._filters);
			}

			this.$emit('filter', {
				value: this.conditions_selected,
			});
			this.commonFilterPopupClose();
		},

		/**
		 * 日期区间选择器开关
		 * @param {Object} option_index
		 */
		dateArrShow(option_index) {
			if (this.date_arr_is_show) {
				this.date_arr_filter_selected = false;
				this.date_arr_is_show = false;
				return;
			}

			this.date_arr_filter_selected = option_index;
			this.date_arr_is_show = true;
		},

		/**
		 * 选中日期区间
		 * @param {Array} dates 日期区间
		 */
		dateArrBindChange(dates) {
			if (this.date_arr_filter_selected === false) {
				return;
			}

			this.date_arr_selected_str = dates[0] + ' 至 ' + dates[1];
			this.date_arr_selected = dates;

			let filter = this.filters[this.actived_index];
			let options = filter.options;

			let option_selected = {};
			options = options.map((option, option_index) => {
				option.is_selected = option_index == this.date_arr_filter_selected;
				if (option.is_selected) {
					option_selected = option;
				}
				return option;
			});

			filter.selected_label = option_selected[filter.options_config.label_field];

			this.conditions_selected[filter.field] = option_selected[filter.options_config.value_field];
			filter.related_fields.forEach((field, i) => {
				this.conditions_selected[field] = dates[i];
			});

			this.$emit('filter', {
				value: this.conditions_selected,
			});
			this.commonFilterPopupClose();
		},

		search() {
			clearInterval(this.search_timer);
			this.search_timer = null;

			this.$base.showLoading('搜索中');

			this.$emit('search', {
				value: this.search_value.trim(),
			});
			this.recordSearchHistory();
			uni.hideLoading();
		},

		recordSearchHistory() {
			if (!this.search_value.trim()) {
				return;
			}
			if (!this._search_history_key) {
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
			this.search_input_focus = !this.search_input_focus;
			this.commonFilterPopupClose();
			this.search_on = !this.search_on;
		},

		commonFilterPopupClose() {
			this.actived_index = -1;
			this.common_filter_popup = false;
		},

		filterPopupOpen() {
			this.common_filter_popup = true;
		},
	},
	watch: {
		//延时搜索
		search_value(value) {
			let i = 1;
			if (this.search_timer) {
				clearInterval(this.search_timer);
			}
			this.search_timer = setInterval(() => {
				++i;
				if (i == 4) {
					if (this.search_value.trim()) {
						this.search();
					} else {
						this.$emit('search', {
							value: this.search_value.trim(),
						});
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
	.search-history {
		margin-top: 0rpx !important;

		&-header {
			border-top: 1rpx solid #eee;
			padding-top: 20rpx;
		}
	}
}
</style>
