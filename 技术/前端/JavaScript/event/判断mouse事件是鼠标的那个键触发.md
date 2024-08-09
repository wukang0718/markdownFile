# 判断mouse事件是鼠标的那个键触发

通过 `MouseEvent`的 `button` 属性判断

| 值   | 描述     |
| ---- | -------- |
| 0    | 鼠标左键 |
| 1    | 鼠标滚轮 |
| 2    | 鼠标右键 |

```javascript
onMousedown = (event) => {
  switch event.button:
  case 0:
    return "鼠标左键"
  case 1:
    return "鼠标滚轮"
  case 2:
    return "鼠标右键"
}
```