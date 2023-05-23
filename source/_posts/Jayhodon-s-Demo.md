---
title: Jayhodon's Demo
date: 2022-05-03 11:34:32
tags: Demo
---
### 将canvas所绘制的图像转成图片格式保存到本地

1. **首先base64图片格式一般都是以下形式：**
   <!-- ‘data:image/jpeg;base64, […base64编码]’ -->

2. **当我们在使用canvas绘图完成后可使用 .toDataURL( )来得到所绘制图像的base64编码数据形式**![toDataURL](../../../../img/baseToImg_1.jpg)

3. **当然也可以将图片格式转成base64格式**![imgToBase](../../../../img/baseToImg_2.jpg)

4. 输出指定文件夹处理
   首先我们需要使用引入第三方工具类，即是

   ```javascript
   const fs = require(‘fs’); 
   ```

![Save](../../../../img/baseToImg_3.jpg)