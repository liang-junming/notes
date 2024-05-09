# Python 语法要点思维导图（含举例）

## 数据类型
- 基本数据类型
  - 整型 (int)
    ```python
    age = 25  # 整型变量
    ```
  - 浮点型 (float)
    ```python
    height = 1.75  # 浮点型变量
    ```
  - 布尔型 (bool)
    ```python
    is_tall = True  # 布尔型变量
    ```
  - 字符串 (str)
    ```python
    name = "Alice"  # 字符串变量
    ```
- 复合数据类型
  - 列表 (list)
    ```python
    fruits = ["apple", "banana", "cherry"]  # 列表
    ```
  - 元组 (tuple)
    ```python
    dimensions = (10, 20, 30)  # 元组
    ```
  - 集合 (set)
    ```python
    prime_numbers = {2, 3, 5, 7}  # 集合
    ```
  - 字典 (dict)
    ```python
    person = {"name": "Bob", "age": 30}  # 字典
    ```

## 控制流
- 条件语句 (if-elif-else)
  ```python
  score = 85
  if score >= 90:
      print("Excellent!")
  elif score >= 80:
      print("Good!")
  else:
      print("Need improvement.")
  ```
- 循环语句
  - for 循环
    ```python
    for i in range(5):
        print(i)
    ```
  - while 循环
    ```python
    i = 0
    while i < 5:
        print(i)
        i += 1
    ```
- 循环控制语句
  - break
    ```python
    for i in range(10):
        if i == 5:
            break
        print(i)
    ```
  - continue
    ```python
    for i in range(10):
        if i % 2 == 0:
            continue
        print(i)
    ```
  - pass
    ```python
    while True:
        pass  # 无限循环
    ```

## 函数
- 定义函数 (def)
  ```python
  def greet(name):
      return f"Hello, {name}!"
  ```
- 参数传递
  - 位置参数
    ```python
    def add(a, b):
        return a + b
    ```
  - 关键字参数
    ```python
    def add(a, b, c=None):
        return a + b + c
    ```
  - 默认参数
    ```python
    def add(a, b=10):
        return a + b
    ```
  - 可变参数
    ```python
    def add(*numbers):
        return sum(numbers)
    ```
- 返回值 (return)
  ```python
  def calculate_area(radius):
      return 3.14 * radius ** 2
  ```
- 匿名函数 (lambda)
  ```python
  square = lambda x: x ** 2
  ```

## 模块和包
- 导入模块 (import)
  ```python
  import math  # 导入整个模块
  from math import sqrt  # 导入特定函数
  ```
- 导入特定函数或类
  ```python
  from datetime import datetime  # 导入datetime类
  ```
- 创建和使用包
  ```python
  # 包的创建涉及目录结构和__init__.py文件
  ```

## 异常处理
- try-except 块
  ```python
  try:
      result = 10 / 0
  except ZeroDivisionError:
      print("Cannot divide by zero!")
  ```
- 抛出异常 (raise)
  ```python
  raise ValueError("This is an error message.")
  ```
- 异常传播 (raise from)
  ```python
  try:
      # some code that may raise an exception
  except SomeException as e:
      raise e from "Additional context."
  ```

## 文件操作
- 打开文件 (open)
  ```python
  with open("file.txt", "r") as file:
      content = file.read()
  ```
- 读取文件 (read, readline, readlines)
  ```python
  with open("file.txt", "r") as file:
      first_line = file.readline()
  ```
- 写入文件 (write, writelines)
  ```python
  with open("new_file.txt", "w") as file:
      file.write("Hello, World!")
  ```
- 文件上下文管理器 (with)
  ```python
  with open("file.txt", "r") as file:
      content = file.read()
  ```

## 类和对象
- 定义类 (class)
  ```python
  class Person:
      def __init__(self, name, age):
          self.name = name
          self.age = age
  ```
- 实例化对象
  ```python
  person = Person("Alice", 25)
  ```
- 构造方法 (__init__)
  ```python
  def __init__(self, name, age):
      self.name = name
      self.age = age
  ```
- 类方法 (classmethod) 和 静态方法 (staticmethod)
  ```python
  @classmethod
  def get_class_name(cls):
      return cls.__name__
  
  @staticmethod
  def add(a, b):
      return a + b
  ```
- 继承
  ```python
  class Student(Person):
      pass
  ```
- 多态
  ```python
  def say_hello(entity):
      entity.hello()
  
  class Person:
      def hello(self):
          print("Hello, I'm a person!")
  
  class Robot:
      def hello(self):
          print("Hello, I'm a robot!")
  
  person = Person()
  robot = Robot()
  say_hello(person)
  say_hello(robot)
  ```

## 装饰器
- 定义装饰器 (def)
  ```python
  def decorator(func):
      def wrapper():
          print("Something is happening before the function is called.")
          func()
          print("Something is happening after the function is called.")
      return wrapper
  ```
- 使用装饰器 (@Before and After function calls)
  ```python
  @decorator
  def say_hello():
      print("Hello!")
  ```
- 带参数的装饰器
  ```python
  def decorator_with_args(arg1, arg2):
      def decorator(func):
          def wrapper():
              print(f"Decorator argument 1: {arg1}")
              print(f"Decorator argument 2: {arg2}")
              func()
          return wrapper
      return decorator
  
  @decorator_with_args("value1", "value2")
  def say_hello():
      print("Hello!")
  ```

## 迭代器和生成器
- 迭代器 (iterator)
  ```python
  numbers = iter([1, 2, 3])
  next(numbers)  # 返回第一个元素
  ```
  - 迭代器协议
    ```python
    class MyNumbers:
        def __init__(self):
            self.current = 0
        def __iter__(self):
            return self
        def __next__(self):
            if self.current < 3:
                current_value = self.current
                self.current += 1
                return current_value
            else:
                raise StopIteration
    ```
  - next()
    ```python
    my_numbers = MyNumbers()
    next(my_numbers)
    ```
- 生成器 (generator)
  ```python
  def count_up_to(max):
      count = 1
      while count <= max:
          yield count
          count += 1
  
  for number in count_up_to(5):
      print(number)
  ```
  - yield
    ```python
    def count_up_to(max):
        count = 1
        while count <= max:
            yield count  # 生成器发送值
            count += 1
    ```

## 常用内置函数
- len()
  ```python
  my_list = [1, 2, 3]
  length = len(my_list)  # 返回列表长度
  ```
- range()
  ```python
  for i in range(5):
      print(i)
  ```
- print()
  ```python
  print("Hello, World!")
  ```
- input()
  ```python
  user_input = input("Please enter your name: ")
  ```
- max(), min()
  ```python
  numbers = [10, 20, 30, 40, 50]
  max_num = max(numbers)  # 返回最大值
  min_num = min(numbers)  # 返回最小值
  ```
- sum(), sorted()
  ```python
  numbers = [10, 20, 30, 40, 50]
  total = sum(numbers)  # 计算总和
  sorted_numbers = sorted(numbers)  # 返回排序后的列表
  ```
  
## 列表推导式
  - 基本列表推导式
    ```python
    squares = [x**2 for x in range(6)]
    ```
  - 复杂列表推导式
    ```python
    even_squares = [x**2 for x in range(10) if x % 2 == 0]
    ```

## 虚拟环境
  - 创建虚拟环境 (venv)
    ```bash
    python -m venv myenv
    ```
  - 激活虚拟环境
    ```bash
    # Windows
    .\myenv\Scripts\activate
    
    # macOS and Linux
    source myenv/bin/activate
    ```

## 代码风格和规范
  - PEP 8
    - 代码风格指南
  - PEP 20
    - The Zen of Python（Python之禅）
  ```
  
  这份思维导图通过具体的代码示例，详细解释了Python编程的基本概念和常用结构。希望这些例子能够帮助你更好地理解和应用Python语法。
  ```
