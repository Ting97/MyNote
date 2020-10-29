## Solution4 ObjectToJSON

### 一、题目描述

​	将一个对象转成JSON格式的字符串：压缩前的字符串和没压缩前的字符串。

### 二、前期准备

#### 2.1 思路

总结一共需要几种类型，然后不同类型的对象按照不同的方式转化成字符串。再根据格式去调整。

#### 2.2 过程

**猜测**：一共需要判断6种类型：List、Map、[]数组、基本类型、其他引用类型，null。

- List：遍历List，从List中获取每个对象，然后进行判断，存储于 [] 中。

- Map：遍历map中每个entry，以(“key”:value)的形式输出。

- []数组：判断[]数组中每个对象，同List。

- 判断为基本类型、基本类型的包装类型、String：直接输出。

- 判断为其他引用类型：遍历字段，同Map。

- 判断为null，输出null。

##### 如何判断类型

- List：object instanceof List

- Map：object instanceof Map
- []数组：object.getClass().isArray()
- 基本类型：Class.isPrimitive()
  - 包装类型
    - Number：Byte 、Short、Integer、Long、Float、Double。
    - Character：char
    - Boolean：boolean
    - String：String

- 其他引用类型：Object

- null：判断 ==null的情况。

##### 做题流程

	1. 判断对象是否为基本类型，直接输出。
 	2. 判断是否为空，为空直接输出。
 	3. 判断是否为包装类型或者String，直接输出。
 	4. 判断是否为Map，遍历map中每个entry，以(“key”:value)的形式输出，存储于{}中。
 	5. 判断是否为List，遍历List，从List中获取每个对象，存储于 [] 中。
 	6. 判断是否为[]，同List。
 	7. 判断是否为其他类型，遍历字段，同Map。









































