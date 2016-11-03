# Ptyhon 基本语法       

标签（空格分隔）： python

---
###python yield使用
```python
def reverse(data):
    for index in range(len(data)-1, -1, -1):
        yield data[index]
for char in reverse('golf'):
        print char
```
###python字符串相加
```
str = "key:%s=%s" % ("s" , 1024)
str = "str1" + "str2"
str1 = 'str''str2' #两个字符串放在一起，会自动相加
str1 = "str""str2"
a = 'abc' * 2 #等于  abcabc


```



