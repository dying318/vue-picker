由于官方picker是基于index维护默认选中，在复杂场景显得过于繁琐，另外官方picker在老版本安卓浏览器中无法显示（已提issue修复方案，后面应该会解决），所以就自己重新写了一套picker。
目前还不支持日期时间选择，仅用于替代下拉多列联动的数据选择。
## 属性/事件列表:

| 属性/事件 | 必填 |  默认值   |  功能  |
| :-----:  | :-----:  | :-----:  | :-----:  |
| pickerList  | 是 |   []     | picker数据，支持多维，使用`children`存储多维数据 |
| defaultValue |   否   |   []   | 默认值，数组格式，和`pickerList`维数保持一致 |
| columnNum |   否   |   0   | 指定列，不传或者为0表示根据数据动态显示列数 |
| itemRotateDeg |   否   |   15   | 每个选项滚动角度，模拟滚轴效果 |
| beforeSetColumn |   否   |   null   | 每次更新列之前调用方法，参数为当前列数据对应的`pickerList`, 可使用该方法对列数据动态的处理，也可以实现动态获取列数据 |
| customStyle |   否   |   {}   | 目前可以自定义按钮样式，列样式，参数格式见: `customStyle` |
| @confirm |   否   |   null   | 点击确定后触发，参数：`picked` |
| @change |   否   |   null   | 列选项更新的时候触发，参数：`columnIndex`, `columnPicked` |
| @cancel |   否   |   null   | 点击取消或者蒙版触发 |


### pickerList:
```
[
    {
        value: 0,
        label: "纪念币0",
        children: [
            {
                value: 1,
                label: "纪念币01",
                children: [
                    {
                        value: 1,
                        label: "纪念币011"
                    }
                ]
            },
            {
                value: 2,
                label: "纪念币02",
            },
            {
                value: 3,
                label: "纪念币03"
            }
        ]
    },
    {
        value: 1,
        label: "纪念币1",
    },
]
```

### customStyle:
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


## 使用方法：
引入picker不要uni-app自带picker冲突，例：MyPicker.
```
<my-picker :pickerList="" @confirm="">
    <div>点击显示Picker</div>
</my-picker>
```


