python调用自己写的模块，提示“ImportError：No module named ...”的情况。



## package的构造

​	package是一种通过用“带点号的模块名”来构造 Python 模块命名空间的方法。 例如，模块名 `A.B` 表示 `A` 包中名为 `B`的子模块。正如模块的使用使得不同模块的作者不必担心彼此的全局变量名称一样，使用加点的模块名可以使得 NumPy 或 Pillow 等多模块软件包的作者不必担心彼此的模块名称一样。

​	一般来说，在目录下添加`__init__.py`就会被认为是package。







### 跨模块调用

feature模块

testcase中调用的时候添加  `sys.path.append(os.path.dirname(__file__)+ "/../")`即可。





参考链接：

1、https://docs.python.org/zh-cn/3/tutorial/modules.html

2、https://www.jianshu.com/p/61ed747680e2