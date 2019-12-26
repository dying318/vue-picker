<template>
    <div>
        <div @click="showPicker">
            <slot></slot>
        </div>
        <div ref="picker" class="picker-pop" @touchmove.prevent>
            <div class="picker-mask" @click="cancel" v-show="show"></div>
            <div class="picker-panel" :class="{'picker-panel-translate': show}">
                <div class="picker-action">
                    <p class="cancel" @click="cancel" :style="customStyle.cancel">取消</p>
                    <p class="confirm" @click="confirm" :style="customStyle.confirm">确定</p>
                </div>
                <div class="picker-content">
                    <div v-for="(column, columnIndex) in columns" class="picker-column" :style="customStyle.column[columnIndex]"
                         :data-column="columnIndex"
                         @touchstart="touchstart" @touchmove="touchmove">
                        <div class="scroll-wrapper">
                            <div class="top-cover"></div>
                            <div class="bottom-cover"></div>
                            <view class="scroll-list" :animation="column.animationData">
                                <div v-for="(data, itemIndex) in column.pickerList" :key="data.value" >
                                    <div class="picker-item" :style="pickerItemStyle(column.pickedIndex, itemIndex)">
                                        {{data.label}}
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
            customStyle: {
                value: Object,
                default() {
                    return {
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
                }
            },
        },
        data() {
            return {
                show: false,
                columns: [],
                pickerItemHeight: Math.floor(68 * screen.width / 750),
                startScrollTop: 0,
                lastPickedIndex: 0,
                scrollingColumnIndex: 0,
            }
        },
        watch: {
            defaultValue() {
                this.init()
            }
        },
        mounted() {
        },
        methods: {
            init() {
                this.setColumn(0, this.pickerList)
            },
            showPicker() {
                this.init()
                if (this.inited) {
                    this.show = true
                } else {
                    let $picker = this.$refs.picker
                    document.body.appendChild($picker)
                    setTimeout(() => {
                        this.show = true
                    }, 20)
                    this.inited = true
                }
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
                    value: column.pickerList[column.pickedIndex].value,
                    label: column.pickerList[column.pickedIndex].label,
                }
            },
            touchstart(e) {
                this.scrollingColumnIndex = e.currentTarget.dataset.column
                this.startScrollTop = e.changedTouches[0].clientY
                this.lastPickedIndex = this.columns[this.scrollingColumnIndex].pickedIndex
            },
            touchmove(e) {
                let scrollDistance = this.startScrollTop - e.changedTouches[0].clientY
                let scrollIndex = Math.round(scrollDistance/this.pickerItemHeight)
                let tempPickerIndex = this.lastPickedIndex + scrollIndex
                let column = this.columns[this.scrollingColumnIndex]
                if (column.pickedIndex !== tempPickerIndex
                    && tempPickerIndex >= 0 && tempPickerIndex < column.pickerList.length) {
                    column.pickedIndex = tempPickerIndex
                    this.$emit('change', column.index, this.columnPickedInfo(column))
                    this.scrollColumn(column)
                }
            },
            setColumn(columnIndex, pickerList) {
                if (columnIndex === 5 || (this.columnNum > 0 && columnIndex >= this.columnNum)) {
                    // limit max 5 columns
                    return
                }
                pickerList = pickerList || []
                let columnPickerList = pickerList
                if (this.beforeSetColumn) {
                    columnPickerList = this.beforeSetColumn(columnIndex, columnPickerList)
                }
                if (columnPickerList.length < 1) {
                    if (this.columnNum === 0) {
                        this.$delete(this.columns, columnIndex)
                        this.clearChildrenColumns(columnIndex, true)
                        return
                    } else if (columnIndex < this.columnNum) {
                        this.clearChildrenColumns(columnIndex)
                    } else {
                        return
                    }
                }

                let currentColumn = this.columns[columnIndex] || {}
                let column = {
                    index: columnIndex,
                    pickerList: columnPickerList,
                    pickedIndex: 0,
                }
                column.pickedIndex = Math.min(currentColumn.pickedIndex, column.pickerList.length - 1) || 0 // 使得下级column的index维持在当前选择位置
                let defaultValue = (this.defaultValue && this.defaultValue[columnIndex]) || false
                if (currentColumn.pickedIndex === undefined && defaultValue !== false) {
                    column.pickerList.map((pickerItem, index) => {
                        if (pickerItem.value == defaultValue) {
                            column.pickedIndex = index
                        }
                    })
                }

                this.scrollColumn(column)
                this.$set(this.columns, columnIndex, column)
            },
            scrollColumn(column) {
                this.setColumn(column.index + 1, column.pickerList[column.pickedIndex].children)

                let translateY = column.pickedIndex * this.pickerItemHeight
                column.animationData = uni.createAnimation({
                    duration: 200,
                    timingFunction: 'linear',
                }).translateY(-translateY).step().export()
            },
            clearChildrenColumns(columnIndex, isRemove = false) {
                if (isRemove) {
                    this.columns.filter(column => {
                        return column.index < columnIndex
                    })
                } else {
                    this.columns.map(column => {
                        if (column.index > columnIndex) {
                            this.setColumn(column.index, [])
                        }
                    })
                }
            },
            pickerItemStyle(pickedIndex, itemIndex) {
                let distance = Math.abs(pickedIndex - itemIndex)
                if (distance <= 3) {
                    return {
                        transform: 'rotateX(' + distance * this.itemRotateDeg + 'deg)'
                    }
                } else {
                    return {}
                }
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
                    line-height: 32rpx;
                    height: 32rpx;
                    font-size: 32rpx;
                }
                .confirm {
                    color: #1CABEB;
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
