#### 如何让表格居中

```css
.table{
    margin: auto;
}
```

#### 表格单双边框问题

```css
/*双边框*/
/*因为表和th/ td元素有独立的边界*/
table, th, td
{
    border: 1px solid black;
}
/*单边框*/
/*border-collapse 属性设置表格的边框是否被折叠成一个单一的边框或隔开*/
table
{
    border-collapse:collapse;
}
table,th, td
{
    border: 1px solid black;
}
```

