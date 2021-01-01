# 将Python编译成二进制

## 安装依赖

```bash
pip install Cython
```

## 准备程序

待编译的源文件hello.py

```python
def main():
    print("hello world")
```

配置文件setup.py

```python
from distutils.core import setup
from Cython.Build import cythonize

setup(
    ext_modules=cythonize("hello.py")
)
```

## 编译

将上面两个文件复制到linux某个目录下，然后执行

```bash
python setup.py build_ext --inplace
```

最后生成了hello.so文件

## 执行

```bash
python -c "from hello import main; main()"
```
