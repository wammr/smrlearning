# BW OJ  8.2思考

## Reverse

### junior math

结合题给代码可知，dword_602064的值应该为0

```C
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  printf("Please input your flag:");
  __isoc99_scanf("%32s", &byte_602080);
  sub_4005D6();
  sub_40063F();
  sub_4006A8();
  sub_400711();
  sub_40077A();
  sub_4007E3();
  sub_40084C();
  sub_4008B5();
  sub_400918();
  sub_400981();
  sub_4009EA();
  sub_400A53();
  sub_400ABC();
  sub_400B25();
  if ( dword_602064 )
    puts("Wrong!");
  else
    puts("Right!");
  return 0LL;
}
```

再点开最后一个函数sub_400B25()可知，dword_605064与v0按位或的值为0，且v0为布尔值，那v0只能为0，说明两个不等式必须都不满足，以此类推，前面的函数也如此

```C
int64 sub_400B25()
{
  _BOOL4 v0; // edx
  __int64 result; // rax

  v0 = (byte_60208D - 54321) * byte_60208D != -5638568 || (byte_60208F - 54321) * byte_60208F != -6774500;
  result = v0 | (unsigned int)dword_602064;
  dword_602064 |= v0;
  return result;
}
```

由此就得到了几组一元二次方程，写python脚本将方程解出：

```python
from sympy import symbols, Eq, solve

# 定义符号变量 x
x = symbols('x')

#定义方程列表
equations = [
    Eq((x - 147) * x, -4590),  #0
    Eq((x - 999) * x, -96228), #1
    Eq((x - 147) * x, -4850),  #2
    Eq((x - 999) * x, -92288),  #3
    Eq((x - 1414) * x,-158793), #4
    Eq((x - 214) * x,-10153), #5
    Eq((x - 1111) * x,-51024), #6
    Eq((x - 1010) * x,-99789), #7
    Eq((x - 741) * x,-45764), #8
    Eq((x - 114) * x,-1805), #9
    Eq((x - 6666) * x,-714713),#A
    Eq((x - 2333) * x,-118612), #B
    Eq((x - 6666) * x,-759800),#C
    Eq((x - 917) * x, -84552),  #D
    Eq((x - 1020) * x, -68256),  #E
    Eq((x - 54321) * x, -6774500),  # E
]

flag=''
# 解方程
for equation in equations:
    solutions = solve(equation, x)
    for sol in solutions:
        if sol >= 48  and sol <= 126:
            flag+=chr(sol)
            #print(chr(sol))

print(flag)
```

得到Flag：flag{G0oD_m4thH}

