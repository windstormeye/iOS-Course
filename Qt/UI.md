
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
