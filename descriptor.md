# 描述符对象
## 概要
python规定，如果一个类包含`__get__`、`__set__`、`__delete__`，三个中的任意一个，则表示该类为一个描述符类，由该类生成的对象被称为描述符。

```py
object.__get__(self, instance, owner)
```
调用此方法以获取所有者类的属性（类属性访问）或该类的实例的属性（实例属性访问）。所有者是指实例的类，而 实例 是指被用来访问属性的实例，如果是所有者被用来访问属性时则为 None。此方法应当返回（计算出的）属性值或是引发一个 AttributeError 异常。

```py
object.__set__(self, instance, value)
```
调用此方法以设置 instance 指定的所有者类的实例的属性为新值 value。

```py
object.__delete__(self, instance)
```
调用此方法以删除 instance 指定的所有者类的实例的属性。

## 描述符调用

总的说来，描述器就是具有“绑定行为”的对象属性，其属性访问已被描述器协议中的方法所重载，包括 `__get__()`, `__set__()` 和 `__delete__()`。如果一个对象定义了以上方法中的任意一个，它就被称为描述器。

属性访问的默认行为是从一个对象的字典中获取、设置或删除属性。例如，a.x 的查找顺序会从 `a.__dict__['x']` 开始，然后是 `type(a).__dict__['x']`，接下来依次查找 type(a) 的上级基类，不包括元类。

描述器发起调用的开始点是一个绑定 a.x。参数的组合方式依 a 而定:

**直接调用**
最简单但最不常见的调用方式是用户代码直接发起调用一个描述器方法: `x.__get__(a)`。

**实例绑定**
`type(a).__dict__['x'].__get__(a, type(a))`

**类绑定**
`A.__dict__['x'].__get__(None, A)`

**超绑定**
如果 a 是 super 的一个实例，则绑定 `super(B, obj).m()` 会在 `obj.__class__.__mro__` 中搜索 B 的直接上级基类 A 然后通过以下调用发起调用描述器: `A.__dict__['m'].__get__(obj, obj.__class__)`。

需要注意的是，从实例绑定中可以看到，只有将属性定义在类层次，被赋值为描述器时，python才会自动调用`__get__`、`__set__`等函数，其根本原因在于直接与实例做绑定的属性python无法去判断出是否是一个描述器，或者说将它判断为一个描述器在大部分场景下并不实用。
```py
class Descriptor(object):
    def __init__(self):
        self.data = {}
    def __get__(self, ins, owner):
        print '__get__'
        return self.data[ins]
    def __set__(self, ins, val):
        print '__set__'
        self.data[ins] = val

class Test(object):
    x = Descriptor()
    y = None
    def __init__(self):
        self.y = Descriptor()

########结果#######
>>> a = Test()
>>> a.x = 1
__set__
>>> a.x
__get__
1
>>> a.y = 1
>>> a.y
1


>>> a = Test()
>>> a.y.__set__(a, 1)
__set__
>>> a.y.__get__(a, None)
__get__
1
```
从上面的代码可以看出，描述符必然要包含一个字典属性，其保存的是每个实例与其值的映射关系。
从结果中可以看出来是符合预期的。