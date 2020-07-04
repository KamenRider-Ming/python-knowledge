# 模块

## 模块加载顺序
加载某一个模块时，仅且在第一次倒入时执行模块里面的内容，然后缓存在当前模块的全局符号表中。

1. 从内置模块中查找，比如sys, os等。
2. 依次遍历`sys.path`中包含的目录，查找目录中是否包含该模块，`sys.path`的目录顺序：当前目录，`PYTHONPATH`环境变量，pip安装目录，`sys.path`可被更改。

**技巧**
1. 使用`sys.path`来快速确认python的依赖库的安装地址和二进制文件运行地址。
2. 使用该命令`PYTHONPATH=./ python Celery/bin/test.py`可以在python项目中正常运行加载单个模块，尤其是只想运行某一个测试样例时，该操作最便捷方便。

## 模块文件
1. **py**
python 源码脚本文件

2. **pyc**
脚本文件编译得到的字节码, 是二进制文件，python文件经过编译器编译之后的文件，不安全，容易被反编译。
可以提高模块文件加载速度，所有模块都会默认生成pyc文件，着属于python编译器内部一种优化机制，这也是为什么所有python开源库或者框架暴露出来的入口文件内容都很简单，基本不包含项目逻辑，其中之一的原因就是如果以加载模块的形式调用的话会生成一个pyc文件，所以我们写脚本时，理论上入口脚本应该越简单越好。

3. **pyo**
脚本文件开启优化编译选项（`-O`）或者(`-OO`)编译得到的字节码，是二进制文件，编译器优化编译后的文件，不安全，容易被反编译；可以通过`python -O test.py`或者``python -OO test.py``生成，其中`-O`会优化掉断言，而`--OO`则会同时优化掉doc-string。
需要清楚认知的一点是，只有test.py所引入的模块（或递归引入）才会生成pyo，本身test.py不会生成一个pyo，如果需要生成的话，请使用`compileall`模块。

4. **pyd**
基本的Windows DLL文件，python的动态链接库，不易被反编译的二进制文件，是一个商业化python工具的安全使用方案之一。

5. **so**
linux环境下的动态链接库，不易被反编译的二进制文件，是一个商业化python工具的安全使用方案之一。

## 包
只有当文件夹下包含有一个`__init__.py`，python才会将该目录识别成一个可导入的包。

### 引入`__init__.py`
#### 从包中导入 *时做限制
```py
test.py
import requests
import string
from pkg import *

目录结构
/
    test.py
    pkg/
        requests.py
        string.py
        test.py
        test_v2.py
        ...
```
**问题**

1. 如果不对pkg使用`import *`的操作加以限制，就会出现我们导入的requests和string库会与pkg导入的requests和string库冲突，从而导致使用异常。
2. 如果我们的pkg的文件夹下有很多模块，很明显使用`import *`直接的结果就是性能降低。

**解决**

如果一个包的 `__init__.py` 代码定义了一个名为 `__all__`属性 的列表，它会被视为在遇到 `from pkg import *` 时应该导入的模块名列表；如果没有`__all__`属性，则`from pkg import *`与`import pkg`等价，它只确保导入了包 pkg （可能运行任何在 `__init__.py` 中的初始化代码），然后导入包中定义的任何名称。这包括 `__init__.py` 定义的任何名称，它还包括由之前的 import 语句显式加载的包的任何子模块。

#### 比较样例
**目录结构**
```py
/
    test.py
    pkg/
        __init__.py
        sub_module1.py
        sub_module2.py
```

**当`__init__.py`为空的情况**
```py
import pkg
from pkg import *
```
第一个语句产生的效果是：由于`__init__.py`中没有任何可运行的内容和被导入的子模块，所以没有任何效果。
第二个语句产生的效果是：游戏`__init__.py`中没有`__all__`属性，所以会尝试处理`__init__.py`文件，但是`__init__.py`没有任何内容，所以该语句没有任何效果。
所以上述两个导入语句的效果一样，没啥用。

**当`__init__.py`文件中内容如下**
```py
import sub_module2

__all__=['sub_module1']

class Test(object):
    name='Test'
```
```py
import pkg
from pkg import *
```
第一个语句产生的效果是：`pkg.sub_module2`和`pkg.Test`可用，但是`pkg.sub_module1`不可用，原因是如果只导入pkg，那么import会忽略掉`__all__`，当作没有`__all__`时加载`__init__.py`文件，所以直接导入一个包，等于将包内的`__init__.py`导入：`import pkg == import pkg.__init__`。
第二个语句产生的效果是：sub_module1可用，但是sub_module2和Test都没有被导入，原因是当存在`__all__`时，`import *`操作虽然依旧会执行`__init__.py`文件，但是最终只会导入`__all__`指定的内容。

