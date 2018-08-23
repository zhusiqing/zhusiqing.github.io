---
layout:     post
title:      html转canvas，canvas转pdf
subtitle:   js,npm,html,canvas,pdf
date:       2018-08-22
author:     siqing
header-img: img/post-bg-debug.png
catalog: true
tags:
    - js
    - npm
    - canvas
    - pdf
---


# html转canvas

## html2canvas

1. 安装

``` bash
npm install html2canvas
```

2. 简单使用

``` js
import html2canvas from 'html2canvas';

// element 是 dom元素
html2canvas(element).then(canvas => {
    // canvas元素
    // 将canvas插入到页面
    document.body.appendChild(canvas);
})
```

3. 拓展使用

``` js
import html2canvas from 'html2canvas';

// element 是 dom元素
html2canvas(element, {
    scale: 2 //放大一倍，使图像清晰一点
}).then(canvas => {
    // 下面可以对生成的canvas进行一系列操作
    // 获取到canvas对象
    const ctx = canvas.getContext('2d');
    // 生成数据
    const pageData = canvas.toDataURL('image/jpeg', 1.0);
    
})
```

# canvas转pdf

## jsPDF

1. 安装

``` bash
npm install jspdf
```

2. 简单使用

    - 文字生成PDF
    ```js
    import jspdf from 'jspdf';
    
    // 默认a4大小，竖直方向，mm单位的PDF
    const doc = new jspdf();
    
    // 添加文本
    // 后面2个是x和y轴的偏移量
    doc.text('添加的文字', 10, 10);
    // 保存
    doc.save('a4.pdf');
    ```
    
    - 文字生成PDF

    ```js
    import jspdf from 'jspdf';
    
    // 默认a4大小，竖直方向，mm单位的PDF
    // 方向，单位，尺寸格式
    // const doc = new jspdf('l', 'pt', [205, 115]);
    const doc = new jspdf('l', 'pt', 'a5');
    
    // 将图片转化为dataUrl
    const imageData = 'data:image/png;base54,idsfsdf...';
    
    // 添加图片
    // 第3，4参数：距离左上角x和y轴偏移的位置 
    // 第5，6参数：生成图片的宽高
    doc.addImage(imageData, 'PNG', 0, 0, 205, 115);
    // 保存
    doc.save('a4.pdf');
    ```

# html转pdf

## html2canvas + jspdf

```js
import html2canvas from 'html2canvas';
import jspdf from 'jspdf'

html2canvas(element, {
    scale: 2 //放大一倍，使图像清晰一点
}).then(canvas => {
    // 下面可以对生成的canvas进行一系列操作
    // 获取到canvas对象
    const ctx = canvas.getContext('2d');
    // 生成数据
    const pageData = canvas.toDataURL('image/jpeg', 1.0);
    // 图片的大小
    const pdfWidth = 417;
    // a5 纸大小 [418, 594]
    // a4 纸大小 [595.28, 841.89]
    const pdf = new jspdf('p', 'pt', 'a5');
    pdf.addImage(pageData, 'JPEG', 0, 50, pdfWidth, pdfWidth / canvas.width * canvas.height);
    // 保存
    pdf.save('html转成的PDF.pdf');
})
```
