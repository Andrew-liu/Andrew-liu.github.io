title: Matplotlib入门指北
date: 2015-05-15 20:22:39
tags: Python
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

#Matplotlib简介

Matplotlib是一个Python工具箱，用于科学计算的数据可视化。借助它，Python可以绘制如Matlab和Octave多种多样的数据图形。最初是模仿了Matlab图形命令, 但是与Matlab是相互独立的.


> 通过Matplotlib中简单的接口可以快速的绘制2D图表

<!--more-->

#初试Matplotlib

Matplotlib中的pyplot子库提供了和matlab类似的绘图API.

```python
import matplotlib.pyplot as plt   #导入pyplot子库

plt.figure(figsize=(8, 4))  #创建一个绘图对象, 并设置对象的宽度和高度, 如果不创建直接调用plot, Matplotlib会直接创建一个绘图对象
plt.plot([1, 2, 3, 4])  #此处设置y的坐标为[1, 2, 3, 4], 则x的坐标默认为[0, 1, 2, 3]在绘图对象中进行绘图, 可以设置label, color和linewidth关键字参数
plt.ylabel('some numbers')  #给y轴添加标签, 给x轴加标签用xlable
plt.title("hello");  #给2D图加标题
plt.show()  #显示2D图
```


#基础绘图


##绘制折线图

与所选点的坐标有关

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python


import numpy as np
import matplotlib.pyplot as plt 

x = [0, 1, 2, 4, 5, 6]
y = [1, 2, 3, 2, 4, 1]

plt.plot(x, y, '-*r')  # 虚线, 星点, 红色
plt.xlabel("x-axis")
plt.ylabel("y-axis")
plt.show()
```

**更改线的样式**查看[plot函数参数设置](http://matplotlib.org/1.4.3/api/pyplot_api.html#matplotlib.pyplot.plot)


##多线图

> 只需要在plot函数中传入多对x-y坐标对就能画出多条线

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python
import numpy as np
import matplotlib.pyplot as plt 

x = [0, 1, 2, 4, 5, 6]
y = [1, 2, 3, 2, 4, 1]
z = [1, 2, 3, 4, 5, 6]

plt.plot(x, y, '--*r', x, z, '-.+g')
plt.xlabel("x-axis")
plt.ylabel("y-axis")
plt.title("hello world")
plt.show()
```



##柱状图

```py
# -*- coding: utf-8 -*-
#!/usr/bin/env python
import numpy as np
import matplotlib.pyplot as plt 

x = [0, 1, 2, 4, 5, 6]
y = [1, 2, 3, 2, 4, 1]
z = [1, 2, 3, 4, 5, 6]

plt.bar(x, y)
plt.xlabel("x-axis")
plt.ylabel("y-axis")
plt.show()
```


##子图

`subplot()`函数指明numrows行数, numcols列数, fignum图个数. 图的个数不能超过行数和列数之积

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python
import numpy as np
import matplotlib.pyplot as plt 

x = [0, 1, 2, 4, 5, 6]
y = [1, 2, 3, 2, 4, 1]
z = [1, 2, 3, 4, 5, 6]

plt.figure(1)
plt.subplot(211)
plt.plot(x, y, '-+b')

plt.subplot(212)
plt.plot(x, z, '-.*r')
plt.show()
```

##文本添加

当需要在图片上调价文本时需要使用`text()`函数, 还有`xlabel(), ylabel(), title()函数`

> text()函数返回`matplotlib.text.Text`, [函数详细解释](http://matplotlib.org/1.4.3/api/pyplot_api.html#matplotlib.pyplot.text)

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python
import numpy as np
import matplotlib.pyplot as plt 

x = [0, 1, 2, 4, 5, 6]
y = [1, 2, 3, 2, 4, 1]

plt.plot(x, y, '-.*r') 
plt.text(1, 2, "I'm a text")  //前两个参数表示文本坐标, 第三个参数为要添加的文本
plt.show()

```


#图例简介


legend()函数实现了图例功能, 他有两个参数, 第一个为样式对象, 第二个为描述字符

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python
import numpy as np
import matplotlib.pyplot as plt 

line_up, = plt.plot([1,2,3], label='Line 2')
line_down, = plt.plot([3,2,1], label='Line 1')
plt.legend(handles=[line_up, line_down])
plt.show()
```


或者调用set_label()添加图例

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python
import numpy as np
import matplotlib.pyplot as plt 

line, = plt.plot([1, 2, 3])
line.set_label("Label via method")
plt.legend()
plt.show()
```

**同时对多条先添加图例**

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python
import numpy as np
import matplotlib.pyplot as plt 

line1, = plt.plot([1, 2, 3])
line2, = plt.plot([3, 2, 1], '--b')
plt.legend((line1, line2), ('line1', 'line2'))
plt.show()
```

更多图例设置可以参考[官方图例教程](http://matplotlib.org/users/legend_guide.html)

#参考链接

- [matplotlib-1.4.3官网文档](http://matplotlib.org/1.4.3/users/intro.html)
