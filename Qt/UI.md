
## 鼠标

## onActivChanged

该事件只要鼠标光标在非目标 `Item` 区按下即可收到信号，与之前的 `onClicked` 这种鼠标光标按下再抬起才接收到信号的事件不同。


### 悬浮
```qml
MouseArea{
    anchors.fill: parent
    cursorShape: Qt.PointingHandCursor
}
```

`cursorShape` 可以修改鼠标光标移动到 `Item` 上时的样式

## 布局

### GridView

使用 `gridView` 时需要给对应的 delegate 包一层 `Component`，这样才能在其中拿到对应数据源子项的 model，否则会提示找不到 model。

```qml
Component {
    id: item
    ImageBrowserItemView {
        id: itemView
        coverUrl: asset.fileUrl
    }
}

GridView {
    id: gridView
    model: viewModel
    delegate: item
    cellWidth: 80
    cellHeight: gridView.height
    anchors.fill: parent
    }
```