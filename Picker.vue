<template>
    <div>
        <div @click.prevent.stop="showPicker">
            <slot></slot>
        </div>
        <div ref="picker" class="picker-pop" v-show="show" @touchmove.prevent.stop>
            <div class="picker-mask" @click="cancel"></div>
            <div class="picker-panel" :class="{'picker-panel-translate': show}">
                <div class="picker-action">
                    <p class="cancel" @click="cancel" :style="pickerStyle.cancel">取消</p>
                    <p class="confirm" @click="confirm" :style="pickerStyle.confirm">确定</p>
                </div>
                <div class="picker-content">
                    <div class="picker-column" v-for="(column, columnIndex) in columns" :key="columnIndex"
                         :style="pickerStyle.column[columnIndex]" :data-column="columnIndex"
                         @touchstart="touchstart" @touchmove="touchmove" @touchend="touchend">
                        <div class="scroll-wrapper">
                            <div class="top-cover"></div>
                            <div class="bottom-cover"></div>
                            <view class="scroll-list" :animation="column.animationData">
                                <div v-for="(data, itemIndex) in column.pickerList" :key="itemIndex" >
                                    <div class="picker-item"
                                        <!-- #ifdef H5-->
                                        :style="pickerItemStyle(column.pickedIndex, itemIndex)"
                                        <!-- #endif-->
                                        >
                                        {{data[pickerKey.label]}}
                                    </div>
                                </div>
                            </view>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        props: {
            pickerList: {
                value: Array,
                require: true,
                default() {
                    return []
                }
            },
            pickerKey: {
                value: Object,
                default() {
                    return {
                        value: 'value',
                        label: 'label',
                        children: 'children'
                    }
                }
            },
            pickerStyle: {
                value: Object,
                default() {
                    return {
                        cancel: {},
                        confirm: {},
                        column: []
                    }
                }
            },
            defaultValue: {
                value: Array,
                default() {
                    return []
                }
            },
            columnNum: {
                value: Number,
                default: 0
            },
            itemRotateDeg: {
                value: Number,
                default: 15
            },
            beforeSetColumn: {
                value: Function,
                default: null
            },
            speedUpRatio: {
                value: Number,
                default: 1
            },
        },
        data() {
            return {
                show: false,
                reactModel: true,
                columns: [],
                pickerItemHeight: Math.floor(68 * uni.getSystemInfoSync().windowWidth / 750),
                startScrollTop: 0,
                startPickedIndex: 0,
                scrollingColumnIndex: 0,
            }
        },
        watch: {
            pickerList() {
                this.init()
            },
            defaultValue() {
                this.init()
            },
        },
        computed: {
            pickerItemStyle() {
                return function(pickedIndex, itemIndex) {
                    let distance = Math.abs(pickedIndex - itemIndex)
                    if (distance <= 3) {
                        return {
                            transform: 'rotateX(' + distance * this.itemRotateDeg + 'deg)'
                        }
                    } else {
                        return {}
                    }
                }
            }
        },
        mounted() {
        },
        methods: {
            init() {
                if (Array.isArray(this.pickerList[0])) {
                    this.pickerList.forEach((pickerList, index) => {
                        this.setColumn(index, pickerList)
                    })
                    this.reactModel = false;
                } else {
                    this.setColumn(0, this.pickerList)
                }

            },
            showPicker() {
                this.init()
                if (this.inited) {
                    this.show = true
                } else {
                    // #ifdef H5
                    let $picker = this.$refs.picker
                    document.getElementsByTagName('uni-page')[0].appendChild($picker)
                    // #endif
                    setTimeout(() => {
                        this.show = true
                    }, 20)
                    this.inited = true
                }
                this.$emit('click') // 传递click事件
            },
            confirm() {
                let picked = {index: [], value: [], label: [], indexes: [],values: [], labels: []}
                for (let column of this.columns) {
                    let columnPicked = this.columnPickedInfo(column)
                    if (columnPicked) {
                        picked.index = columnPicked.index
                        picked.value = columnPicked.value
                        picked.label = columnPicked.label

                        picked.indexes.push(columnPicked.index)
                        picked.values.push(columnPicked.value)
                        picked.labels.push(columnPicked.label)
                    } else {
                        picked.indexes.push(null)
                        picked.values.push(null)
                        picked.labels.push(null)
                    }
                }
                this.$emit('confirm', picked)
                this.hide()
            },
            cancel() {
                this.$emit('cancel')
                this.hide()
            },
            hide() {
                this.show = false
            },
            columnPickedInfo(column) {
                if (column.pickerList.length < 1) {
                    return null
                }
                return {
                    index: column.pickedIndex,
                    value: column.pickerList[column.pickedIndex][this.pickerKey.value],
                    label: column.pickerList[column.pickedIndex][this.pickerKey.label],
                }
            },
            touchstart(e) {
                this.scrollingColumnIndex = e.currentTarget.dataset.column
                this.startScrollTop = e.changedTouches[0].clientY
                this.startPickedIndex = this.columns[this.scrollingColumnIndex].pickedIndex

                this.columns[this.scrollingColumnIndex].scrollEventQueue = [{
                    index: this.startPickedIndex,
                    time: +new Date()
                }]
            },
            touchmove(e) {
                let scrollDistance = this.startScrollTop - e.changedTouches[0].clientY
                let scrollIndex = Math.round(scrollDistance/this.pickerItemHeight)
                let column = this.columns[this.scrollingColumnIndex]
                let currentPickedIndex = column.pickedIndex
                this.setColumnIndex(column, this.startPickedIndex + scrollIndex)
                if (column.pickedIndex !== currentPickedIndex) {
                    this.scrollColumn(column, true)
                }
            },
            touchend(e) {
                let column = this.columns[this.scrollingColumnIndex]
                this.scrollColumn(column, false, true)
            },
            setColumn(columnIndex, pickerList) {
                if (columnIndex === 5 || (this.columnNum > 0 && columnIndex >= this.columnNum)) {
                    // limit max 5 columns
                    return
                }
                let columnPickerList = pickerList || []
                if (this.beforeSetColumn) {
                    // 在开始渲染列之前使用钩子动态修改pickerList，注意避免对pickerList修改以保证渲染不污染源数据
                    columnPickerList = this.beforeSetColumn(columnIndex, columnPickerList)
                }
                if (columnPickerList.length < 1) {
                    if (this.columnNum === 0) {
                        // 动态列数，当前列为空，清除后面全部列
                        this.columns = this.columns.filter(column => {
                            return column.index < columnIndex
                        })
                        return
                    } else if (columnIndex < this.columnNum) {
                        // 固定列数，清除下一列，递归清除后面全部列
                        this.setColumn(columnIndex + 1, [])
                    } else {
                        return
                    }
                }

                let currentColumn = this.columns[columnIndex] || {}
                let column = {
                    index: columnIndex,
                    scrollEventQueue: [],
                    pickerList: columnPickerList,
                    pickedIndex: 0,
                }
                this.setColumnIndex(column, currentColumn.pickedIndex || 0) // 使得column的index维持在当前选择位置
                let defaultValue = this.defaultValue && this.defaultValue[columnIndex] !== false ? this.defaultValue[columnIndex] : false
                if (currentColumn.pickedIndex === undefined && defaultValue !== false) {
                    column.pickerList.map((pickerItem, index) => {
                        if (pickerItem[this.pickerKey.value] == defaultValue) {
                            column.pickedIndex = index
                        }
                    })
                }

                this.scrollColumn(column)
                this.$set(this.columns, columnIndex, column)
            },
            setColumnIndex(column, index) {
                index = index < 0 ? 0 : index
                column.pickedIndex = Math.min(index, column.pickerList.length - 1)
            },
            scrollColumn(column, needThrottle = false, needSpeedUp = false) {
                let now = +new Date()
                let lastScrollEvent = column.scrollEventQueue[column.scrollEventQueue.length-1]
                if (needThrottle && lastScrollEvent.time && now < (lastScrollEvent.time + 100)) {
                    return
                }
                let speedUpIndex = 0
                if (needSpeedUp && this.speedUpRatio) {
                    // 模拟惯性效果，在touch事件接触后，根据最后两次滚动事件的速度生成滑动的距离。在touch过程中，保持触摸距离和滚动距离的一致
                    if (column.scrollEventQueue.length > 1) {
                        lastScrollEvent = column.scrollEventQueue[column.scrollEventQueue.length-2]
                    }
                    let speed = (column.pickedIndex - lastScrollEvent.index) / (now - lastScrollEvent.time)
                    speedUpIndex = Math.floor(Math.pow(speed, 2) * 800 * this.speedUpRatio) // 使用二次方曲线放大加速效果，其中效果800为默认调试参数
                    speedUpIndex = speed > 0 ? speedUpIndex : -speedUpIndex;
                    this.setColumnIndex(column, column.pickedIndex + speedUpIndex)
                }

                column.scrollEventQueue.push({
                    index: column.pickedIndex,
                    time: now
                })

                let translateY = column.pickedIndex * this.pickerItemHeight
                column.animationData = uni.createAnimation({
                    duration: 200,
                    timingFunction: 'linear',
                }).translateY(-translateY).step().export()

                if (this.reactModel && column.pickerList[column.pickedIndex]) {
                    this.setColumn(column.index + 1, column.pickerList[column.pickedIndex][this.pickerKey.children])
                }

                this.$emit('change', column.index, this.columnPickedInfo(column))
            }
        },
    };
</script>

<style lang="scss" scoped>
    .picker-pop {
        .picker-mask {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            z-index: 999;
            width: 100vw;
            height: 100vh;
            background-color: rgba(0, 0, 0, .6);
        }

        .picker-panel {
            position: fixed;
            bottom: 0;
            left: 0;
            z-index: 999;
            width: 100%;
            background-color: #fff;
            transform: translate(0, 100%);
            transition: transform .3s;

            .picker-action {
                display: flex;
                position: relative;
                justify-content: space-between;

                &:after {
                    content: '';
                    position: absolute;
                    right: 0;
                    bottom: 0;
                    left: 0;
                    height: 1px;
                    transform: scaleY(.5);
                    background-color: #dedede;
                }

                p {
                    color: #999;
                    padding: 30rpx;
                    line-height: 1;
                    font-size: 36rpx;
                }
                .confirm {
                    color: #007aff;
                }
            }

            .picker-content {
                height: calc(68rpx * 7);
                overflow: hidden;
                position: relative;
                display: flex;

                .picker-column {
                    flex: 1;
                    font-size: 32rpx;
                    overflow: hidden;
                }

                .scroll-wrapper {
                    position: relative;
                    height: calc(68rpx * 7);

                    .top-cover, .bottom-cover {
                        width: 100%;
                        position: absolute;
                        z-index: 1;
                        transform: translateZ(0);
                        height: calc(68rpx * 3);
                        background: linear-gradient(0deg,hsla(0,0%,100%,.3),hsla(0,0%,100%,.9));

                        &:before {
                            content: '';
                            position: absolute;
                            right: 0;
                            bottom: 0;
                            left: 0;
                            height: 1px;
                            transform: scaleY(.5);
                            background-color: #dedede;
                        }
                    }
                    .top-cover {
                        top: 0;
                    }
                    .bottom-cover {
                        bottom: 0;
                        background: linear-gradient(180deg,hsla(0,0%,100%,.3),hsla(0,0%,100%,.9));

                        &:before {
                            top: 0;
                        }
                    }
                    .scroll-list {
                        padding-top: calc(68rpx * 3);

                        .picker-item {
                            text-align: center;
                            font-size: 32rpx;
                            line-height: 68rpx;
                            height: 68rpx;
                            color: #333;
                            white-space: nowrap;
                            overflow: hidden;
                            text-overflow: ellipsis;
                        }
                    }

                }

            }
        }

        .picker-panel-translate {
            transform: translate(0, 0);
        }
    }
</style>
