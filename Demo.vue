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
            @change="changeMultiColumns"
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
            @change="changeCustomMultiColumns"
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
    import MyPicker from './Picker';

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
            changeMultiColumns(index, picked) {
                this.demos.multiColumns.columnPickedIndex = index
                this.demos.multiColumns.columnPicked = picked
            },
            changeCustomMultiColumns(index, picked) {
                this.demos.customMultiColumns.columnPickedIndex = index
                this.demos.customMultiColumns.columnPicked = picked
            },
            addPickerItem(index, pickerList) {
                if (pickerList.length > 0) {
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
