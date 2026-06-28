---
title: Python 经典基础编程 20 例
published: 2026-06-28
description: 收集整理了 20 个经典的 Python 基础编程示例，代码均保持最精简、最易读的状态。
tags:
  - Python
  - 编程
category: 教程
draft: false
---

下面是 20 个经典的 Python 基础编程示例，代码均保持最精简、最易读的状态。

## 1. 判断奇偶数

```python
num = int(input("请输入一个整数: "))
if num % 2 == 0:
    print(f"{num} 是偶数")
else:
    print(f"{num} 是奇数")
```

## 2. 删除列表中的重复数据

```python
ls = [1, 2, 1, 2, 3, 5, 4, 3, 7, 4, 7, 8, 9]

# 保持原有顺序的去重
new_ls = []
for i in ls:
    if i not in new_ls:
        new_ls.append(i)
print("去重后的列表:", new_ls)
```

## 3. 判断子串

```python
def is_substring(str1, str2):
    return str2 in str1

# 测试
print(is_substring("hello world", "world"))  # 输出: True
```

## 4. while 循环倒序打印 10-1

```python
i = 10
while i >= 1:
    print(i, end=" ")
    i -= 1
print()  # 换行
```

## 5. 运煤问题计算

```python
import math

total = 68.5
car1_capacity = 4
car1_times = 3
car2_capacity = 2.5

# 计算剩余的煤量
remaining = total - (car1_capacity * car1_times)
# 计算剩余需要的次数，若有余数则需要向上取整
times = math.ceil(remaining / car2_capacity)
print(f"还需要运送 {times} 次才能运送完。")
```

## 6. 计算最大公约数（GCD）

```python
import math

num1 = int(input("请输入第一个整数: "))
num2 = int(input("请输入第二个整数: "))
print("最大公约数是:", math.gcd(num1, num2))
```

## 7. 判断是否能构成三角形

```python
def is_triangle(a, b, c):
    # 三角形判定条件：任意两边之和大于第三边
    return a + b > c and a + c > b and b + c > a

# 测试
a = float(input("边长a: "))
b = float(input("边长b: "))
c = float(input("边长c: "))
print("是否能构成三角形:", is_triangle(a, b, c))
```

## 8. 计算 1-100 之间的奇数和

```python
odd_sum = 0
for i in range(1, 101):
    if i % 2 != 0:
        odd_sum += i
print("1-100之间的奇数和为:", odd_sum)
```

## 9. 字典的添加与修改

```python
student = {"name": "小明", "age": 18, "score": 92}

# 添加“gender”键值对
student["gender"] = "男"
# 修改“score”的值
student["score"] = 95

print(student)
```

## 10. 将列表元素替换为其长度

```python
fruits = ["apple", "banana", "orange"]

# 列表推导式
new_list = [len(item) for item in fruits]

print(new_list)  # 输出: [5, 6, 6]
```

## 11. 计算圆的直径和面积

```python
import math

r = float(input("请输入圆的半径: "))
diameter = 2 * r
area = math.pi * (r**2)

print(f"圆的直径为: {diameter:.2f}")
print(f"圆的面积为: {area:.2f}")
```

## 12. 字符串反转

```python
def reverse_string(s):
    # 使用切片操作 [::-1] 快速反转
    return s[::-1]

user_str = input("请输入一个字符串: ")
print("反转后的字符串:", reverse_string(user_str))
```

## 13. 向元组中的列表添加元素

```python
tup = (1, 23, 56, "hi", [1, 2])
# 访问元组的最后一个元素并使用 append 方法
tup[-1].append("h")
print(tup)  # 输出: (1, 23, 56, 'hi', [1, 2, 'h'])
```

## 14. for 循环打印 1-50 之间能被 3 整除的数

```python
for i in range(1, 51):
    if i % 3 == 0:
        print(i, end=" ")
print()
```

## 15. 判断闰年

```python
year = int(input("请输入年份: "))
if (year % 4 == 0 and year % 100 != 0) or (year % 400 == 0):
    print(f"{year} 是闰年")
else:
    print(f"{year} 不是闰年")
```

## 16. 输出 1 到 n 之间所有的质数

```python
def print_primes(n):
    for num in range(2, n + 1):
        is_prime = True
        for i in range(2, int(num**0.5) + 1):
            if num % i == 0:
                is_prime = False
                break
        if is_prime:
            print(num, end=" ")
    print()

n = int(input("请输入一个正整数n: "))
print(f"1到{n}之间的质数有:")
print_primes(n)
```

## 17. 判断正数还是负数

```python
num = float(input("请输入一个数: "))
if num > 0:
    print("这是一个正数")
elif num < 0:
    print("这是一个负数")
else:
    print("这是0")
```

## 18. 计算列表中所有元素的平均值

```python
nums = [23, 15, 7, 48, 32, 91]
average = sum(nums) / len(nums)
print(f"平均值为: {average:.2f}")
```

## 19. 计算阶乘乘积（20×19×…×3）

```python
def calculate_product():
    result = 1
    for i in range(3, 21):
        result *= i
    return result

print("计算结果为:", calculate_product())
```

## 20. while 语句计算 1-100 之间所有数的和

```python
total_sum = 0
i = 1
while i <= 100:
    total_sum += i
    i += 1
print("1-100之间所有数的和为:", total_sum)
```
