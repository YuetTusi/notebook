
## 方向

|类名|CSS|
|---|---|
|flex-row|flex-direction: row;|
|flex-row-reverse|flex-direction: row-reverse;|
|flex-col|flex-direction: column;|
|flex-col-reverse|flex-direction: column-reverse;|

## 增长&收缩率

|样式|CSS|
|---|---|
|flex-1|flex: 1 1 0%;|
|flex-auto|flex: 1 1 auto;|
|flex-initial|flex: 0 1 auto;|
|flex-none|flex: none;|


精确值使用`flex-[*]`如：`flex-[2]`，`flex-[2_2_0%]`。

以下定义一个3列布局，第1列占比1，第3列占比2，中间不撑满：

```html
<div class="flex w-full text-center">
		<div class="flex-1 bg-green-400">1</div>
		<div class="flex-none bg-pink-400">2</div>
		<div class="flex-[2] bg-orange-400">3</div>
</div>
```

![[de6a9c3f-5799-4511-80c3-d74fdff27ad3.png]]

## 主轴对齐

|样式|CSS|
|---|---|
|justify-normal|justify-content: normal;|
|justify-start|justify-content: flex-start;|
|justify-end|justify-content: flex-end;|
|justify-center|justify-content: center;|
|justify-between|justify-content: space-between;|
|justify-around|justify-content: space-around;|
|justify-evenly|justify-content: space-evenly;|
|justify-stretch|justify-content: stretch;|

## 交叉轴对齐

|样式|CSS|
|---|---|
|items-start|align-items: flex-start;|
|items-end|align-items: flex-end;|
|items-center|align-items: center;|
|items-baseline|align-items: baseline;|
|items-stretch|align-items: stretch;|


## 换行

|类名|Properties|
|---|---|
|flex-wrap|flex-wrap: wrap;|
|flex-wrap-reverse|flex-wrap: wrap-reverse;|
|flex-nowrap|flex-wrap: nowrap;|