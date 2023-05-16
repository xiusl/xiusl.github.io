---
title: datastruct-pycook
date: 2019-01-12 16:34:30
categories:
- technology
- Python
tags:
- cookbook
- datastruct
---
python cookbook datastruct
<!--more-->

- 解压序列赋值给多个变量
  - 变量的个数和序列元素的个数不匹配
  - 只解压一部分
    ```py
    data = ["abc", 10, 22.1, (2018, 12, 12)]
    _, shares, prices, _= data
    print(shares)  # 10
    print(prices)  # 22.1
    ```
- 解压可迭代对象赋值给多个变量
  - 星号(\*)表达式
    ```py
    record = ('Dave', 'dave@example.com', '213-123-1421', '312-423-1511')
    name, email, *phone_numbers = record
    print(name)  # Dave
    print(email)  # deve@example.com
    print(phone_numbers)  # ['213-123-1421', '312-423-1511']
    ```
  - 星号表达式解压出来的永远是**列表**类型
  - 迭代元素为可变长元组的序列
    ```py
    records = [
        ('foo', 1, 2),
        ('bar', 'hello'),
        ('foo', 3, 5)
    ]

    def do_foo(x, y):
        return x+y

    def do_bar(str):
        print(str)

    for tag, *args in records:
        if tag == 'foo':
            do_foo(*args)
        elif tag == 'bar':
            do_bar(*args)
    ```
  - 星号表达式操作字符chuan
    ```py
    line = 'nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false'
    uname, *fields, homedir, sh = line.split(':')
    print(uname)  # 'nobody'
    print(homedir)  # '/var/empty'
    print(sh)  # '/usr/bin/false'
    ```
  - 解压并丢弃一些元素， `_` or `ign(ignore)`
    ```py
    record = ('abc', 12, 23.1, (12, 12, 2018))
    name, *_, (*_, year) = record
    print(name)  # 'abc'
    print(year)  # 2012
    ```
- 保留最后 N 个元素
  - 保留有限历史记录 `collections.deque`
    ```py
    from collections import deque

    def search(lines, pattern, history=5):
        previous_lines = deque(maxlen=history)
        for line in lines:
            if pattern in line:
                yield line, previous_lines
            previous_lines.append(line)

    if __name__ == '__main__':
        with open(r'../../cookbook/somefile.txt') as f:
            for line, prevlines in search(f, 'python', 5):
                for pline in prevlines:
                    print(pline, end='')
                print(line, end='')
                print('-' * 20)
    ```
  - `deque(maxlen=N)` 会创建一个固定大小的队列，当新的元素加入并且队列已满的时候，最老的元素会自动被移除
    ```py
    q = deque(maxlen=3)
    q.append(1)
    q.append(2)
    q.append(3)
    print(q)  # deque([1, 2, 3], maxlen=3)
    q.append(4)
    print(q)  # deque([2, 3, 4], maxlen=4)
    ```
  - `deque()` 不设置最大长度，将会创建一个无限大小队列，可以在两端增删元素
    ```
    q = deque()
    q.append(1)
    q.append(2)
    q.append(3)
    print(q)  # deque([1, 2, 3])
    q.appendleft(4)
    print(q)  # deque([4, 1, 2, 3])
    q.pop()
    print(q)  # deque([4, 1, 2])
    q.popleft()
    print(q)  # deque([1, 2])
    ```

- 查找最大或最小的 N 个元素
  - `heapq` 模块的 `nlargest()` 和 `nsmallest()` 两个函数
  - 函数可以接受一个关键字参数，用于复杂的数据结构
    ```py
    import heapq
    nums = [1, 8, 3, 23, 4, -1, 12, 34, 54, 3]
    print(heapq.nlargest(3, nums))  # [54, 34, 23]
    print(heapq.nsmallest(3, nums))  # [-1, 1, 3]


    portfolio = [
        {'name': 'IBM', 'shares': 100, 'price': 91.1},
        {'name': 'AAPL', 'shares': 50, 'price': 543.22},
        {'name': 'FB', 'shares': 200, 'price': 21.09},
        {'name': 'HPQ', 'shares': 35, 'price': 31.75},
        {'name': 'YHOO', 'shares': 45, 'price': 16.35},
        {'name': 'ACME', 'shares': 75, 'price': 115.69}
    ]
    cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
    expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])
    ```
  - 函数的底层实现是将集合进行堆排序再放入列表中
  - 堆排序的特征是 heap[0] 始终是最小的元素
  - ...
  - 查找元素个数相对比较少的时候，函数 nlargest() 和 nsmallest() 是很合适的
  - 如果你仅仅查找唯一的最小或最大的元素的话，使用 min() 和 max() 会更快一些
  - 如果查找元素的个数和集合的大小接近的话，通常先排序这个集合然后在使用切片操作会更快

- 实现优先级队列
  - 使用 heapq 实现简单的优先级队列
    ```py
    import heapq

    class PriorityQueue:
        def __init__(self):
            self._queue = []
            self._index = 0

        def push(self, item, priority):
            heapq.heappush(self._queue, (-priority, self._index, item))
            self._index += 1

        def pop(self):
            return heapq.heappop(self._queue)[-1]

    class Item:
        def __init__(self, name):
            self.name = name

        def __repr__(self):
            return 'Item({!r})'.format(self.name)

    q = PriorityQueue()
    q.push(Item('foo'), 1)
    q.push(Item('bar'), 5)
    q.push(Item('spam'), 4)
    q.push(Item('grok'), 1)
    print(q.pop())
    print(q.pop())
    print(q.pop())
    print(q.pop())
    ```
- 字典排序
  - 使用 `collections` 模块中的 `OrderedDict` 类，控制一个字典中元素的顺序
    ```py
    from collections import OrderedDict

    d = OrderedDict()
    d['foo'] = 1
    d['bar'] = 2
    d['spam'] = 3
    d['grok'] = 4

    for key in d:
        print(key, d[key])
    ```
  - 当需要序列化或编码成其他格式的映射的时候，`OrderedDict` 是非常有用的。例如精确控制 `JSON` 编码后字段的顺序
    ```py
    import json
    json.dumps(d)
    ```
  - `OrderedDict` 的内部维护着一个很具键插入顺序排序的双向链表。每次一个新元素插入的时候，它会放到链表的尾部。对一个已经存在的键的重复赋值不会改变键的顺序
  - 一个 `OrderedDict` 的大小是一个普通字典的两倍，因为它内部维护这另一个链表，当在处理大量数据的时候需要考虑内存消耗
- 字典的运算
  - 字典中执行运算操作，最值、排序等
  - 使用 `zip()` 函数将键和值翻转
    ```py
    prices = {
        'ACME': 45.23,
        'AAPL': 612.78,
        'IBM': 205.55,
        'HPQ': 37.20,
        'FB': 10.75
    }
    min_price = min(zip(prices.values(), prices.keys()))
    max_price = max(zip(prices.values(), prices.keys()))
    ```
  - 类似的可以使用 `sorted()` 函数来排序字典数据
  - 需要注意 `zip()` 函数创建的是一个只能访问一次的迭代器
    ```py
    prices_and_names = zip(prices.values(), prices.keys())
    print(min(prices_and_name))
    print(max(Prices_and_name)) # ValueError max arg is an empty sequence
    ```
  - 在一个字典上执行普通的数学运算，只会作用在键上
    ```py
    min(prices) # 'AAPL'
    max(prices) # 'IBM'
    ```
  - 当考虑使用 `values()` 方法解决的时候，无法获取对应键的信息
    ```py
    min(prices.values()) # 10.75
    max(prices.values()) # 612.78
    ```
  - 使用 `min()` 和 `max()` 函数中的参数 `key` 可以获取对应的键的信息，但又无法一次性获取对应值得信息
    ```py
    min(prices, key=lambda k: prices[k]) # 'FB'
    max(prices, key=lambda k: prices[k]) # 'AAPL'

    min_value = prices[min(prices, key=lambda k: pirces[k])]
    ```
  - 在使用(值,键)对的时，多个实体拥有相同值得时候，键会决定返回结果
    ```py
    prices = {'AAA': 45.23, 'ZZZ', 45.23}
    min(zip(prices.values(), prices.keys())) # (45.23, 'AAA')
    max(zip(prices.values(), prices.keys())) # (45.23, 'ZZZ')
    ```
- 查找两个字典的相同点
  - 通过集合操作来获取字典相同的键、相同的值
    ```py
    a = {'x': 1, 'y': 2, 'z': 3}
    b = {'w': 10, 'x': 11, 'y': 2}
    print(a.keys() & b.keys()) # {'x', 'y'}
    print(a.keys() - b.keys()) # {'z'}
    print(a.items() & b.items()) # {('y', 2)}
    ```
  - 修改或过滤字典元素
    ```py
    c = { key:a[key] for key in a.keys() - {'z', 'w'}}
    print(c) # {'x': 1, 'y': 2}
    ```
  - 字典的键可以直接执行集合的运算操作，不需要转换为 `set`
  - 字典的值是不支持直接进行集合操作的，需要转换为 `set` 才可以
- 删除序列相同元素并保持顺序
  - 序列上的值都是 `hashable` 类型，可以使用集合或者生成器来解决
    ```py
    def dedupe(items):
        seen = set()
        for item in items:
            if item not in seen:
                yield item
                seen.add(item)

    a = [1, 5, 2, 1, 9, 1, 5, 10]
    print(list(dedupe(a))  # [1, 5, 2, 9, 10]
    ```
  - 元素不可以哈希时
    ```py
    def dedupe(items, key=None):
        seen = set()
        for item in items:
            val = item if key is None else key(item)
            if val not in seen:
                yield item
                seen.add(val)

    a = [{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 1,'y': 2}, {'x': 2, 'y': 4}]
    print(list(dedupe(a, key=lambda d: (d['x'], d['y']))))
    # [{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]
    print(list(dedupe(a, key=lambda d: d['x'])))
    # [{'x': 1, 'y': 2}, {'x': 2, 'y': 4}]
    ```
- 命名切片
  - 硬编码切片下标（固定字符串提取指定位置的字段）
    ```py
    record = '...............100.........513.25.........'
    cost = int(record[15:18]) * float(record[27:33])

    SHARES = slice(15, 18)
    PRICE = slice(27, 33)
    cost = int(record[SHARES]) * float(record[PRICE])
    ```
  - 内置的 `slice()` 函数创建一个切片对象，可以被用在任何允许使用切片的地方
    ```py
    items = [0, 1, 2, 3, 4, 5, 6]
    a = slice(2, 4)
    print(items[2:4])  # [2, 3]
    print(items[a])  # [2, 3]
    items[a] = [10, 11]
    print(items[a])  # [10, 11]
    del items[a]
    print(items)  # [0, 1, 4, 5, 6]
    ```
  - 切片对象的属性可以获取相应的信息
    ```py
    a = slice(5, 50, 2)
    print(a.start)  # 5
    print(a.stop)  # 50
    print(a.step)  # 2
    ```
  - 切片对象的 `indices(size)` 方法可以将它映射到一个确定大小的序列上
    ```py
    s = 'HelloWorld'
    print(a.indices(len(s)))  # (5, 10, 2)
    for i in range(*a.indices(len(s))):
        print(s[i])
    # W
    # r
    # d
    ```

