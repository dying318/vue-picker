由于官方picker是基于index维护默认选中，在复杂场景显得过于繁琐，另外官方picker在老版本安卓浏览器中无法显示（已提issue修复方案，后面应该会解决），所以就自己重新写了一套picker。
目前还不支持日期时间选择，仅用于替代下拉多列联动的数据选择。
## 属性/事件列表:

| 属性/事件 | 必填 |  默认  |  功能  |
| :-----:  | :-----:  | :-----:  | :-----  |
| pickerList  | 是 |   []     | picker数据，支持多维，使用`children`存储多维数据 |
| pickerKey |   否   |   {value: 'value',lable:'label',children: 'children'}   | 用于指定用户自定义数据字段名和默认字段名对应关系 |
| pickerStyle |   否   |   {}   | 目前可以自定义按钮样式，列样式，参数格式见: `pickerStyle` |
| defaultValue |   否   |   []   | 默认值，数组格式，和`pickerList`维数保持一致 |
| columnNum |   否   |   0   | 指定列，不传或者为0表示根据数据动态显示列数，最多限制5列 |
| itemRotateDeg |   否   |   15   | 每个选项滚动角度，模拟滚轴效果, 注意：该效果目前只支持H5 |
| beforeSetColumn |   否   |   null   | 每次更新列之前调用方法，参数：`columnIndex`,`pickerList`，其中`pickerList`对应当前列数据， 可使用该方法对列数据动态的处理，也可以用于ajax动态获取列数据。注意：如果未设置`columnNum`，只有`beforeSetColumn`返回空值或者超出5列才会停止渲染新的列 |
| speedUpRatio |   否   |   1   | 增加了滑动惯性特性，该参数可以设置惯性的速率比例。默认1为预设速率，设置为0则没有惯性。 |
| @confirm |   否   |   null   | 点击确定后触发，参数：`picked` |
| @change |   否   |   null   | 列选项更新的时候触发，参数：`columnIndex`, `columnPicked` |
| @cancel |   否   |   null   | 点击取消或者蒙版触发 |


### pickerList:
```
// 联动多列
[
    {
        label: '选项1',
        value: 1,
        children: [
            {
                label: '选项11',
                value: 11,
                children: [
                    {label: '选项111', value: 111},
                ]
            },
        ]
    },
]

// 非联动多列
[
    [
        {label: '选项1', value: 1},
        {label: '选项2', value: 2},
        {label: '选项3', value: 3},
    ],
    [
        {label: '选项1', value: 1},
        {label: '选项2', value: 2},
        {label: '选项3', value: 3},
    ],
]

```

### pickerStyle:
```
{
    cancel: {
        color: '#999',
    },
    confirm: {
        color: '#1CABEB',
    },
    column: [
        {flex: 1},
        {flex: 1},
        {flex: 3},
    ]
}
```

### picked:
```
{
    index: 1
    indexes: [3, 2, 1]
    label: "1953年2分"
    labels: ["3版", "2分", "1953年2分"]
    value: 116
    values: [4, 115, 116]
}
```

### columnPicked:
```
{
    index: 1
    label: "1953年2分"
    value: 116
}
```


## Demo：
引入picker不要uni-app自带picker冲突，例：MyPicker.

#### 单列：
```
<my-picker :picker-list="" @confirm="confirm">
</my-picker>
```

#### 固定多列：
```
<my-picker
    column-num="3"
    :picker-list=""
    @change="change"
    @confirm="confirm">
</my-picker>
```

#### 完整参数演示：
```
<my-picker
    column-num="3"
    :picker-list=""
    :picker-style=""
    :picker-key="{value: 'id', label: 'title', children: 'sub'}"
    :before-set-column="addPickerItem"
    :default-value="[2,21,212]"
    :item-rotate-deg="20"
    @change="change"
    @confirm="confirm">
</my-picker>
```

#### 非固定列：
```
<my-picker
    :picker-list=""
    @confirm="confirm">
</my-picker>
```

#### 完整Demo
```
<template>
    <view class="container">
        <my-picker :picker-list="singleColumnPickerList" @confirm="confirm('singleColumn', $event)">
            <button class="button" type="primary">单列</button>
        </my-picker>
        <view class="picked-result" v-if="demos.singleColumn.picked">
            <h4>单列picker选择结果：</h4>
            <pre v-html="JSON.stringify(demos.singleColumn.picked, null, 4)"></pre>
        </view>

        <my-picker
            :picker-list="multiColumnsPickerList"
            column-num="3"
            @change="change('multiColumns', ...arguments)"
            @confirm="confirm('multiColumns', $event)">
            <button class="button" type="primary">3列联动</button>
        </my-picker>
        <view class="picked-result" v-if="demos.multiColumns.picked">
            <h4>3列联动选择结果：</h4>
            <pre v-html="JSON.stringify(demos.multiColumns.picked, null, 4)"></pre>
        </view>
        <view class="picked-result" v-else-if="demos.multiColumns.columnPicked">
            <h4>多列联动第{{demos.multiColumns.columnPickedIndex + 1}}列更新：</h4>
            <pre v-html="JSON.stringify(demos.multiColumns.columnPicked, null, 4)"></pre>
        </view>

        <my-picker
            column-num="3"
            :picker-list="multiColumnsWithCustomKeyPickerList"
            :picker-style="pickerStyle"
            :picker-key="{value: 'id', label: 'title', children: 'sub'}"
            :before-set-column="addPickerItem"
            :default-value="[2,21,212]"
            :item-rotate-deg="20"
            @change="change('customMultiColumns', ...arguments)"
            @confirm="confirm('customMultiColumns', $event)">
            <button class="button" type="primary">完整自定义参数联动</button>
        </my-picker>
        <view class="picked-result" v-if="demos.customMultiColumns.picked">
            <h4>完整自定义参数联动选择结果：</h4>
            <pre v-html="JSON.stringify(demos.customMultiColumns.picked, null, 4)"></pre>
        </view>
        <view class="picked-result" v-else-if="demos.customMultiColumns.columnPicked">
            <h4>完整自定义参数联动第{{demos.customMultiColumns.columnPickedIndex + 1}}列更新：</h4>
            <pre v-html="JSON.stringify(demos.customMultiColumns.columnPicked, null, 4)"></pre>
        </view>

        <my-picker
            :picker-list="multiColumnsPickerList"
            @confirm="confirm('dynamicColumns', $event)">
            <button class="button" type="primary">非固定列联动</button>
        </my-picker>
        <view class="picked-result" v-if="demos.dynamicColumns.picked">
            <h4>非固定列联动选择结果：</h4>
            <pre v-html="JSON.stringify(demos.dynamicColumns.picked, null, 4)"></pre>
        </view>

        <my-picker
            :picker-list="independentMultiColumnsPickerList"
            @confirm="confirm('independentMultiColumns', $event)">
            <button class="button" type="primary">多列非联动</button>
        </my-picker>
        <view class="picked-result" v-if="demos.independentMultiColumns.picked">
            <h4>多列非联动选择结果：</h4>
            <pre v-html="JSON.stringify(demos.independentMultiColumns.picked, null, 4)"></pre>
        </view>
    </view>
</template>

<script>
    import MyPicker from '../../components/Picker';

    export default {
        components: {MyPicker},
        data() {
            return {
                demos: {
                    singleColumn: {
                        picked: null,
                    },
                    multiColumns: {
                        picked: null,
                        columnPicked: null,
                        columnPickedIndex: null,
                    },
                    customMultiColumns: {
                        picked: null,
                        columnPicked: null,
                        columnPickedIndex: null,
                    },
                    dynamicColumns: {
                        picked: null,
                        columnPicked: null,
                        columnPickedIndex: null,
                    },
                    independentMultiColumns: {
                        picked: null,
                        columnPicked: null,
                    },
                },
                pickerStyle: {
                    cancel: {
                        color: '#999',
                        'font-size': '32rpx'
                    },
                    confirm: {
                        color: 'green',
                        'font-size': '32rpx'
                    },
                    column: [
                        {flex: 1},
                        {flex: 1},
                        {flex: 2},
                    ]
                },
                singleColumnPickerList: [
                    {label: '选项1', value: 1},
                    {label: '选项2', value: 2},
                    {label: '选项3', value: 3},
                    {label: '选项4', value: 4},
                    {label: '选项5', value: 5},
                    {label: '选项6', value: 6},
                    {label: '选项7', value: 7},
                    {label: '选项8', value: 8},
                    {label: '选项9', value: 9},
                ],
                multiColumnsPickerList: [
                    {
                        label: '选项1',
                        value: 1,
                        children: [
                            {
                                label: '选项11',
                                value: 11,
                                children: [
                                    {label: '选项111', value: 111},
                                    {label: '选项112', value: 112},
                                    {label: '选项113', value: 113},
                                    {label: '选项114', value: 114},
                                    {label: '选项115', value: 115},
                                ]
                            },
                            {
                                label: '选项12',
                                value: 12,
                                children: [
                                    {label: '选项121', value: 121},
                                    {label: '选项122', value: 122},
                                    {label: '选项123', value: 123},
                                    {label: '选项124', value: 124},
                                    {label: '选项125', value: 125},
                                ]
                            },
                            {label: '选项13', value: 13},
                            {label: '选项14', value: 14},
                            {label: '选项15', value: 15},
                        ]
                    },
                    {
                        label: '选项2',
                        value: 2,
                        children: [
                            {
                                label: '选项21',
                                value: 21,
                                children: [
                                    {label: '选项211', value: 211},
                                    {label: '选项212', value: 212},
                                    {label: '选项213', value: 213},
                                    {label: '选项214', value: 214},
                                    {label: '选项215', value: 215},
                                ]
                            },
                            {
                                label: '选项22',
                                value: 22,
                                children: [
                                    {label: '选项221', value: 221},
                                    {label: '选项222', value: 222},
                                    {label: '选项223', value: 223},
                                    {label: '选项224', value: 224},
                                    {label: '选项225', value: 225},
                                ]
                            },
                            {label: '选项23', value: 23},
                            {label: '选项24', value: 24},
                            {label: '选项25', value: 25},
                        ]
                    },
                    {label: '选项3', value: 3},
                    {label: '选项4', value: 4},
                    {label: '选项5', value: 5},
                    {label: '选项6', value: 6},
                    {label: '选项7', value: 7},
                    {label: '选项8', value: 8},
                    {label: '选项9', value: 9},
                ],
                multiColumnsWithCustomKeyPickerList: [
                    {
                        title: '选项1',
                        id: 1,
                        sub: [
                            {
                                title: '选项11',
                                id: 11,
                                sub: [
                                    {title: '选项111', id: 111},
                                    {title: '选项112', id: 112},
                                    {title: '选项113', id: 113},
                                    {title: '选项114', id: 114},
                                    {title: '选项115', id: 115},
                                ]
                            },
                            {
                                title: '选项12',
                                id: 12,
                                sub: [
                                    {title: '选项121', id: 121},
                                    {title: '选项122', id: 122},
                                    {title: '选项123', id: 123},
                                    {title: '选项124', id: 124},
                                    {title: '选项125', id: 125},
                                ]
                            },
                            {title: '选项13', id: 13},
                            {title: '选项14', id: 14},
                            {title: '选项15', id: 15},
                        ]
                    },
                    {
                        title: '选项2',
                        id: 2,
                        sub: [
                            {
                                title: '选项21',
                                id: 21,
                                sub: [
                                    {title: '选项211', id: 211},
                                    {title: '选项212', id: 212},
                                    {title: '选项213', id: 213},
                                    {title: '选项214', id: 214},
                                    {title: '选项215', id: 215},
                                ]
                            },
                            {
                                title: '选项22',
                                id: 22,
                                sub: [
                                    {title: '选项221', id: 221},
                                    {title: '选项222', id: 222},
                                    {title: '选项223', id: 223},
                                    {title: '选项224', id: 224},
                                    {title: '选项225', id: 225},
                                ]
                            },
                            {title: '选项23', id: 23},
                            {title: '选项24', id: 24},
                            {title: '选项25', id: 25},
                        ]
                    },
                    {title: '选项3', id: 3},
                    {title: '选项4', id: 4},
                    {title: '选项5', id: 5},
                    {title: '选项6', id: 6},
                    {title: '选项7', id: 7},
                    {title: '选项8', id: 8},
                    {title: '选项9', id: 9},
                ],
                independentMultiColumnsPickerList: [
                    [
                        {label: '选项1', value: 1},
                        {label: '选项2', value: 2},
                        {label: '选项3', value: 3},
                        {label: '选项4', value: 4},
                        {label: '选项5', value: 5},
                        {label: '选项6', value: 6},
                        {label: '选项7', value: 7},
                        {label: '选项8', value: 8},
                        {label: '选项9', value: 9},
                    ],
                    [
                        {label: '选项1', value: 1},
                        {label: '选项2', value: 2},
                        {label: '选项3', value: 3},
                        {label: '选项4', value: 4},
                        {label: '选项5', value: 5},
                        {label: '选项6', value: 6},
                        {label: '选项7', value: 7},
                        {label: '选项8', value: 8},
                        {label: '选项9', value: 9},
                    ],
                ]
            }
        },
        methods: {
            confirm(type, picked) {
                this.demos[type].picked = picked
            },
            change(type, index, picked) {
                this.demos[type].columnPickedIndex = index
                this.demos[type].columnPicked = picked
            },
            addPickerItem(index, pickerList) {
                if (index === 2 && pickerList.length > 0) {
                    pickerList = [{title: '全部', id: 0}].concat(pickerList)
                }

                return pickerList
            }
        }
    }
</script>

<style lang="scss" scoped>
    .container {
        display: flex;
        flex-direction: column;
        align-items: center;

        height: 100vh;
        padding: 20rpx;

        .button {
            margin-top: 20rpx;
        }

        .picked-result {
            width: 80%;
            margin-top: 20rpx;
            padding: 20rpx;
            background-color: #f3f3f3;
            color: #555;
            font-size: 24rpx;
            line-height: 1;
        }
    }

</style>

```


