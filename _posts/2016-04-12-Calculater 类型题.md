---
layout: post
title: calculator 类型题
date: 2016-04-12
tag: Leetcode
---

计算器类型题的大意是给你一个用string表示的数学计算式，求解它。这类题型的典型解法是stack加上priority。我们来一步步看例题。

##### Basic calculator I
> 给定一个只含有加减与括号的式子，里边的数字都是non-negative的，计算结果

思路：建立两个stack，分别存operators and operands.并且赋予每个operator一个priority。当遇到（ 时，priority + 1， 当遇到 ） 时，priority - 1.如此可以实现括号中先计算的功能。而且当一个operator的priority <= 之前的operators时，就要pop stack中的元素进行计算。最终我们维护的是priority严格递增的stack。

注意，下面的code只能handle比较正规的写法，负数或者括号前的减号前没数字的表达方式没法胜任。

方案一：
```python
class Solution(object):
    def calculate(self, s):
        s = s.replace(' ', '')
        return 0
        operators = []
        operands = []
        priority = 0
        i = 0
        while i < len(s):
            c = s[i]
            if c == '+' or c == '-':
                while operators and operators[-1][1] >= priority:
                    temp = operands.pop()
                    op = operators.pop()[0]
                    operands[-1] = self.cal(operands[-1], op, temp)
                operators.append([c, priority])
            elif c.isdigit():
                num = ord(c) - ord('0')
                while i + 1 < len(s) and s[i+1].isdigit():
                    num = num * 10 + ord(s[i+1]) - ord('0')
                    i += 1
                operands.append(num)
            elif c == '(':
                priority += 1
            elif c == ')':
                priority -= 1
            i += 1


        while operators:
            temp = operands.pop()
            op = operators.pop()[0]
            operands[-1] = self.cal(operands[-1], op, temp)

        return operands[0]

        def cal(self, num1, op, num2):
            if op == '+':
                return num1 + num2
            else:
                return num1 - num2
```

还有一种思路，即每当遇到）时，就把前边（内所有内容都计算出来；每当遇到operator，就把（到此为止的内容计算出来。

方案二：
```python
class Solution(object):
    def calculate(self, s):
        s = s.replace(' ', '')

        operators = []
        operands = []

        i = 0
        while i < len(s):
            c = s[i]
            if c == '+' or c == '-':
                while operators and operators[-1] != '(':
                    temp = operands.pop()
                    op = operators.pop()
                    operands[-1] = self.cal(operands[-1], op, temp)
                operators.append(c)
            elif c.isdigit():
                num = ord(c) - ord('0')
                while i + 1 < len(s) and s[i+1].isdigit():
                    num = num * 10 + ord(s[i+1]) - ord('0')
                    i += 1
                operands.append(num)
            elif c == '(':
                operators.append(c)
            elif c == ')':
                while operators and operators[-1] != '(':
                    temp = operands.pop()
                    op = operators.pop()
                    operands[-1] = self.cal(operands[-1], op, temp)
                operators.pop()
            i += 1


        while operators:
            temp = operands.pop()
            op = operators.pop()
            operands[-1] = self.cal(operands[-1], op, temp)

        return operands[0]

    def cal(self, num1, op, num2):
        if op == '+':
            return num1 + num2
        else:
            return num1 - num2
```

##### Basic calculator II
> 给定一个只含有加减与乘除的式子,里边的数字都是non-negative的，计算结果

思路：依然使用的是stack加上priority的做法。维持priority严格递增的stack。

```python
class Solution(object):
    def calculate(self, s):
        s = s.replace(' ', '')

        operators = []
        operands = []
        priority = 0

        i = 0
        while i < len(s):
            c = s[i]
            if c in ['+', '-', '*', '/']:
                if c in ['+', '-']:
                    priority = 0
                else:
                    priority = 1
                while operators and operators[-1][1] >= priority:
                    num = operands.pop()
                    op = operators.pop()[0]
                    operands[-1] = self.cal(operands[-1], op, num)
                operators.append([c, priority])
            else:
                while i + 1 < len(s) and s[i + 1].isdigit():
                    c = c + s[i + 1]
                    i += 1
                operands.append(int(c))
            i += 1

        while operators:
            num = operands.pop()
            op = operators.pop()[0]
            operands[-1] = self.cal(operands[-1], op, num)

        return operands[-1]

    def cal(self, num1, op, num2):
        if op == '+':
            return num1 + num2
        elif op == '-':
            return num1 - num2
        elif op == '*':
            return num1 * num2
        else:
            return num1 / num2
```

##### Basic calculator III
> 给定一个含有加减乘除与括号的式子，里边的数字都是non-negative的，计算结果

思路：此题是上两题的混合，方案一是采用stack + priority + 递归的做法，遇到括号就把括号内的内容先计算出来再放回去（其实calculator I也可以用递归来做）；方案二是对加减乘除采用priority，而对）采取的是一直pop的做法。其实是calculator I的方案二与calculator II 的结合。

```python
class Solution(object):
    def calculate(self, s):
        s = s.replace(' ', '')

        operators = []
        operands = []
        priority = 0

        i = 0
        while i < len(s):
            c = s[i]
            if c in ['+', '-', '*', '/']:
                if c in ['+', '-']:
                    priority = 0
                else:
                    priority = 1
                while operators and operators[-1][0] != '(' and operators[-1][1] >= priority:
                    num = operands.pop()
                    op = operators.pop()[0]
                    operands[-1] = self.cal(operands[-1], op, num)
                operators.append([c, priority])
            elif c == '(':
                operators.append([c, 0])
            elif c == ')':
                while operators and operators[-1][0] != '(':
                    num = operands.pop()
                    op = operators.pop()[0]
                    operands[-1] = self.cal(operands[-1], op, num)
                operators.pop()
            else:
                while i + 1 < len(s) and s[i + 1].isdigit():
                    c = c + s[i + 1]
                    i += 1
                operands.append(int(c))
            i += 1

        while operators:
            num = operands.pop()
            op = operators.pop()[0]
            operands[-1] = self.cal(operands[-1], op, num)

        return operands[-1]

    def cal(self, num1, op, num2):
        if op == '+':
            return num1 + num2
        elif op == '-':
            return num1 - num2
        elif op == '*':
            return num1 * num2
        else:
            return num1 / num2
```

##### Basic calculator IV
> 给定一个只含有加减乘除，括号与字母的式子，计算结果

```python
class Solution:
    def calc(self, a, b, op):
        if op == '+':
            for k, v in b.items():
                a[k] = a.get(k, 0) + v
            return a
        elif op == '-':
            for k, v in b.items():
                a[k] = a.get(k, 0) - v
            return a
        elif op == '*':
            t = {}
            for k1, v1 in a.items():
                for k2, v2 in b.items():
                    newk = tuple(sorted(k1+k2))
                    t[newk] = t.get(newk, 0) + v1 * v2
            return t

    def basicCalculatorIV(self, expression, evalvars, evalints):
        vars = {n:v for n,v in zip(evalvars, evalints)}
        d = []  # operands
        op = []
        priority = {'(': 0, '+': 1, '-': 1, '*': 2}
        arr = re.findall(r'\(|\)|[a-z]+|[0-9]+|[\+\-\*]', expression)
        for t in arr:
            if t[0].isdigit():
                d.append({tuple():int(t)})
            elif t[0].isalpha():
                if t in vars:
                    d.append({tuple():vars[t]})
                else:
                    d.append({(t,): 1})
            elif t == '(':
                op.append(t)
            elif t == ')':
                while op and op[-1] != '(':
                    d.append(self.calc(d.pop(-2), d.pop(-1), op.pop()))
                op.pop()
            elif t in '+-*':
                if not op or priority[t] > priority[op[-1]]:
                    op.append(t)
                else:
                    while op and priority[t] <= priority[op[-1]]:
                        d.append(self.calc(d.pop(-2), d.pop(-1), op.pop()))
                    op.append(t)
        while op:
            d.append(self.calc(d.pop(-2), d.pop(-1), op.pop()))

        res = []
        for k in sorted(d[0].keys(), key=lambda x: (-len(x), x)):
            v = d[0][k]
            if v != 0:
                if not k:
                    res.append(str(v))
                else:
                    res.append('%s*%s' % (v, '*'.join(k)))
        return res
```