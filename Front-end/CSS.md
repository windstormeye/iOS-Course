# CSS
## 基本
### `<ul>` 实现 `<li>` 等分
```html
<ul id="father">
  <li class="children">第一个</li>
  <li class="children">第二个</li>
  <li class="children">第三个</li>
</ul>
```

```css
#father{
  display:flex;
  width:100%;
  height:10rem;
}
.children{
  flex:1;
  text-align:center;
}
```

### `<ul>` 