## Uniapp列表与筛选/搜索/拼音索引🌌

*简易版文档，仅供交流学习，插件版后续看情况出，欢迎各位大神指导*

### 引入
``` javascript
import ListSelectOnline from '@/components/list-select/list-select-online.vue';

export default {
	components: {
		ListSelectOnline,
	}
}
```

### 使用
``` javascript
<list-select-online pageKey="staff" :filters="filters" :searchPlaceHolder="search_placeHolder" @search="search" @filter="filter">
    
			<block v-if="list.length">
                <Staff v-for="(item, index) in list" :key="index" :staff="item" @tap="select(item)"></Staff>
            </block>

			<no-data v-else :tip="Object.keys(filter_conditions).length || search_value ? '换个条件试试?' : ''"></no-data>
</list-select-online>
```


###  配置

| 属性名            | 类型    | 默认值               | 说明                                     |
| ----------------- | ------- | -------------------- | ---------------------------------------- |
| filters           | Array   | []                   | 筛选选项                                 |
| useSearch         | Boolean | true                 | 是否使用搜索栏                           |
| searchPlaceHolder | String  | 点击搜索             | 搜索栏占位符                             |
| listNone          | Object  | `{tip: '空空如也~'}` | 列表为空时显示文字                       |
| pageKey           | String  | ''                   | 页面key标识 不设置则无法存储页面搜索记录 |



- **筛选选项 filters[].* (Object)**

| 属性名 | 类型 | 默认值 | 说明 |
| ------ | ------------ | ---- | ---- |
| title | String | 必填 | 筛选名 |
| field | String | 必填 | 筛选字段，传给后端的key |
| style | String | 必填 | 筛选样式 |
| show | Object | {type: 'all', result: true, hidden: false, condition_field: '', condition_values: []} | 显示设置 |
| children | Array | 当style为multiplex时，此字段存在 | 子筛选按钮 |
| options | Array | options_config为static时必填 | 筛选选项 |
| options_config | Object | {type: 'static', function: '',ache: true,unshift_all: true, label_field: 'label',value_field: 'value',default: null} | 筛选选项配置 |
| related_fields | Array | 需要时声明 | 自订参数，需要传递多个值时使用，目前在使用日期区间时可用 |

- **筛选选项 filters[].* (Object).style(String)**

| 值 | 说明 |
| ---- | ---- |
| line | 一行一行的 |
| block | 一块一块的 |
| multiplex | 复杂选项的，包含多个子筛选         |
| slot | 自定义，slot name 为 ”slot_filter“ |

- **筛选选项 filters[].* (Object).show(Object)**

| 值               | 默认                       | 说明                                                         |
| ---------------- | -------------------------- | ------------------------------------------------------------ |
| type             | all                        | 类型 except / only / all                                     |
| hidden           | false                      | 是否隐藏, 有些页面需要动态判断是否使用此选项                 |
| condition_field  | ''                         | 条件判断字段，根据condition_field的值来进行判断              |
| condition_values | type为except / only 时必填 | 条件成立值。当type为except时，condition_field的值**不在**condition_values中，则满足条件；当type为only时，condition_field的值**在**condition_values中，则满足条件；当type为all时，全部显示，忽略condition_field与condition_values |

- **筛选选项 filters[].* (Object).options[].*(Object)**

| 值                                  | 默认 | 说明                                                  |
| ----------------------------------- | ---- | ----------------------------------------------------- |
| label (可动态改变)                  | 必填 | 筛选选项名称                                          |
| value (可动态改变)                  | 必填 | 筛选选项值                                            |
| type (需要时声明)                   | 无   | 自己规定选项 [component 为点击触发某个组件]           |
| component (type为component时需声明) | 无   | 触发的组件名称   [pyh-rdtpicker 为时间区间选择器组件] |

- **筛选选项 filters[].* (Object).options_config(Object)**

| 值          | 默认                       | 说明                                                         |
| ----------- | -------------------------- | ------------------------------------------------------------ |
| type        | static                     | 类型 static 固定选项  dynamic 动态加载选项                   |
| function    | type为dynamic 时必填       | 加载选项需要执行的方法, 该方法返回一个Promise                |
| cache       | true，type为dynamic 时有效 | 是否缓存选项，只请求加载选项一次                             |
| unshift_all | true                       | 是否自动在选项第一个补全一个"全部"选项, label为"全部"，value为0 |
| label_field | label                      | 选项的label替换成别的key                                     |
| value_field | value                      | 选项的value替换成别的key                                     |
| default     | null                       | 是否是默认选中项，null为不是                                 |

### 案例



- Ⅰ


```vue
<list-select-online
    pageKey="card"
    :filters="filters"
    :searchPlaceHolder="search_placeHolder"
    :useSearch="user_search"
    @search="search"
    @filter="filter"
>
    <!-- item模板 -->
    <block v-if="list.length"><Card v-for="(item, index) in list" :key="index" :card="item" @tap="select(item)"></Card></block>
    <no-data v-else :tip="Object.keys(filter_conditions).length || search_value ? '换个条件试试?' : ''"></no-data>
</list-select-online>

```

```javascript
data() {
    return {
        init_over: false,
        user_search: true,
        search_placeHolder: '可搜索会员名或手机号',
        filters: [],

        search_value: '',
        filter_conditions: {}, //筛选框选择的筛选条件
        fixed_conditions: {}, //固定筛选 外部传入
    };
 },

//初始化filters      
this.filters = [
    {
        title: '状态',
        field: 'flag', //通过某个字段匹配
        style: 'block',
        options: Object.values(CARD_FLAG),
    },
    {
        title: '卡类型',
        field: 'type', //通过某个字段匹配
        style: 'line',
        options: [
            {
                label: '期限卡',
                value: 1,
            },
            {
                label: '次数卡',
                value: 2,
            },
            {
                label: '储值卡',
                value: 3,
            },
        ],
    },
    {
        title: '支持功能',
        field: 'allow',
        style: 'block',
        options: [
            {
                label: '出入场',
                value: 1,
            },
            {
                label: '团课',
                value: 2,
            },
            {
                label: '私教',
                value: 3,
            },
            {
                label: '储物柜',
                value: 4,
            },
        ],
    },
];

let multiplex_filter = {
    title: '更多',
    style: 'multiplex',
    children: [
        {
            title: '多人共用',
            field: 'support_multiple', //通过某个字段匹配
            style: 'line',
            options: [
                {
                    label: '支持',
                    value: 1,
                },
                {
                    label: '不支持',
                    value: 0,
                },
            ],
        },
        {
            title: '卡名称',
            field: 'category_id',
            style: 'block',
            options_config: {
                type: 'dynamic',
                function: 'getCardCategoryNames',
                cache: true,
                label_field: 'name',
                value_field: 'id',
            },
            options: [],
        },
    ],
};


this.filters.push(multiplex_filter)





select(item) {
    this.$base.navigateTo('./detail?id=' + item.id);
},

search(e) {
    this.search_value = e.value;
    this._initData();//重新加载数据
},

filter(e) {
    this.filter_conditions = e.value;
    this._initData();//重新加载数据
},

//动态加载options的方法
getCardCategoryNames() {
    return new Promise(resolve => {
        this.$go('options/card-category').then(res => {
            resolve(res.data.data || []);
        });
    });
},
```



- Ⅱ

  ```javascript
  一个日期选择选项配置
  let filters = [{
          title: '日期',
          field: 'date_type',
          related_fields: ['date_start', 'date_end'],
          style: 'line',
          options_config: {
              type: 'static',
              default: null,
              unshift_all: true,
          },
          options: [{
                  label: '今日',
                  value: 1,
              },
              {
                  label: '本周',
                  value: 2,
              },
              {
                  label: '本月',
                  value: 3,
              },
              {
                  label: '指定日期段',
                  value: -1,
                  type: 'component',
                  component: 'pyh-rdtpicker',
              },
          ],
      }]
  ```

  





#### 拼音索引

**使用拼音索引需要一次性加载所有数据，在本地进行数据筛选**









#### 引用组件

- <u> _[日期区间picker](https://ext.dcloud.net.cn/plugin?id=435)</u>

