### flex:1 下的文字无法 ellipsis/overflow:scroll

  上层的所有 flex:1 带上 min-width:0(ellipsis)  min-height:0(overflow:scroll)
  
  refers:
  - [javascript - text-overflow: ellipsis and flex - Stack Overflow](https://stackoverflow.com/questions/39472747/text-overflow-ellipsis-and-flex)
  - [html - Text-overflow: ellipsis causes flex item to overflow flex container - Stack Overflow](https://stackoverflow.com/questions/41286912/text-overflow-ellipsis-causes-flex-item-to-overflow-flex-container)
  - [CSS Flexible Box Layout Module Level 1](https://www.w3.org/TR/css-flexbox-1/#intrinsic-item-contributions)

### inline-block 的孩子上方总是有空隙

  父容器设置 font-size:0; 或者首个不包含文字的孩子设置 font-size: 0

  refers:
  [html - display: inline-block leaves gap with respect to height after div element - Stack Overflow](https://stackoverflow.com/questions/27730680/display-inline-block-leaves-gap-with-respect-to-height-after-div-element)
  
### 移除元素之间的空白

[Fighting the Space Between Inline Block Elements | CSS-Tricks](https://css-tricks.com/fighting-the-space-between-inline-block-elements/)

### 宽高等比

padding-top/padding-bottom 设置百分比是根据宽度的，所以设置元素 `padding-top  3/4*100%` 表示宽高比 4:3

### align-self:flex-end 无法置于底部

[css - Flexbox column align self to bottom - Stack Overflow](https://stackoverflow.com/questions/24697267/flexbox-column-align-self-to-bottom#answer-35125244)

给置底元素加上 `margin-top: auto;`
