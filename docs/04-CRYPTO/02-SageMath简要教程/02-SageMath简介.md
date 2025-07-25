# SageMath简介

SageMath是一个Python在数学应用上的一个扩展程序，在掌握Python的基本语法之后SageMath相当于多了很多自带扩展库

### 1. 与Python的不同之处

```bash
type(5)
<class 'sage.rings.integer.Integer'>
```

5的类型不是int，而是整数环上的元素

```bash
5**3        # 表示5的三次方
5^3         # 表示5的三次方，python中表示异或
5^^3        # 表示5异或3
```

### 2. 基本的环，域

1. 整数环：ZZ
2. 有理数环：QQ
3. 实数域：RR
4. 复数域：CC
5. 多项式环：PolynomialRing()

用的最多的是`PolynomialRing()`

```bash
PQ.<x> = PolynomialRing(QQ)
```

PQ：是多项式环自定义的名字

x：变量名

### 3. 基本数论函数

1. 求逆：e模n的逆

> 如果`e`是定义在$Zmod(n)$上的元素，直接`e^-1`即可得到逆元。
> 否则直接使用`inverse_mod(e,n)`，一般情况都是用`Crypto.Util.number`模块中的`inverse(e,n)`

2. 最大公因数：(a,b)的最大公因数

> `gcd(a,b)`

3. 最小公倍数：(a,b)的最小公倍数

> `lcm(a,b)`

4. 模幂：$e^x \mod n$

> 如果`e`是定义在$Zmod(n)$上的元素，直接`e^x`即可，否则使用`pow(e,x,n)`

5. 素数判断：

> ```bash
> x = 7
> x.is_prime()
> # True
> ```
> 我一般用Crypto.Util.number模块中的`isPrime(x)`

6. 阶乘：

> `factorial(x)`

7. 欧拉函数

> ```bash
> euler_phi(n)
> ```
> 求$\phi(n)$

8. 中国剩余定理

> ```bash
> crt([m1,m2],[n1,n2])
> ```

> 求解同余方程组
> $$
> x \equiv m_1 \mod n_1\\
> x \equiv m_2 \mod n_2
> $$

9. 扩展欧几里得算法

> ```bash
> d,x,y = xgcd(a,b)
> ```

> $$
> d = gcd(a,b) = ax + by
> $$
> `gmpy2.gcdext(a,b)`

10. 素数分解

```bash
factor(1024)
# 2^10 
prime_divisors(1024)
# [2]
divisors(1024)
# [1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024]
```

$p_1^{e_1} * p_2^{e_2} ... p_n^{e_n}$

11. 开根

整数域开根
$$
x^m = y
$$
已知$y,m$求$x$

```bash
y = 87^8
y.nth_root(8)
# 87
```

有限域开根
$$
x^m \equiv y \mod N
$$
已知$y,m,N$求$x$

```bash
y = pow(78,888,65537)
x = Zmod(65537)(y).nth_root(888)
print(x)
# 78
```

$$
c \equiv m^e \mod n
$$

`x = Zmod(n)(c).nth_root(e)`

注意：开根有多解，`nth_root`只会返回一个解，如果需要得到所有解，可以多加一个参数

`x = Zmod(65537)(y).nth_root(888,all=True)`

```bash
y = pow(78,888,65537)
x = Zmod(65537)(y).nth_root(888,all=True)
print(x)
# [78, 64289, 19968, 8197, 65459, 1248, 45569, 57340]
```

12. 离散对数

sage实现了多种离散对数的求解方法
$$
y \equiv x^m \mod p
$$


参数说明：求解以`base`为底，`a`的对数；`ord`为`base`的阶（可以缺省），`operation`可以是`+`，`*`，默认是`*`；`bound`是一个区间`(ld,ud)`，需要保证所计算的对数在此区间内

+ `discrete_log(a,base,ord,operation)`

  通用的求离散对数的方法

+ `discrete_log_rho(a,base,ord,operation)`

  求离散对数的Pollard-Rho算法

+ `discrete_log_lambda(a,base,bounds,operation)`

​		求离散对数的Pollard-kangaroo算法（也称lambda算法）

+ bsgs(base,a,bounds,operation)

  大步小步算法

当`operation`为`+`时，一般是应用在椭圆曲线的离散对数

### 4. 线性代数相关函数

+ 矩阵的定义运算

  1. 一般矩阵定义

  ```bash
  # 定义整数环上的矩阵
  A = Matrix(ZZ,[
  [1,2,3],
  [4,5,6],
  [7,8,9]
  ])
  # 模2有限域的矩阵
  B = Matrix(GF(2),A)
  B = Matrix(Zmod(2),A)
  # 向量定义
  v = vector(ZZ,[1,2,3])
  v = vector(GF(2),[1,2,3])
  # 定义矩阵但不初始化:GF(2)上的10 * 10矩阵默认所有元素为0
  M = Matrix(GF(2),10,10)
  ```

  2. 分块矩阵定义：

  $$
  M = \begin{bmatrix}
  A & B\\
  C & D
  \end{bmatrix}
  $$

  ```bash
  A = Matrix(GF(2),10,10)
  B = Matrix(GF(2),10,30)
  C = Matrix(GF(2),5,10)
  D = Matrix(GF(2),5,30)
  M = block_matrix([
  [A,B],
  [C,D]
  ])
  # 矩阵的规模
  ```

  3. 矩阵运算

  ```python
  M1 = Matrix([
      [1,2],
      [1,3]
  ])
  M2 = Matrix([
      [2,1],
      [2,3]
  ])
  M1 + M2
  """
  [3 3]
  [3 6]
  """
  M1 * M2
  """
  [ 6  7]
  [ 8 10]
  """
  ```

  转置：`M.transpose()`

  求逆：`M.inverse()`或者`M^(-1)`

  特征值：`M.eigenvalues()`

  特征向量（右）：`M.eigenvalues_right()`

  行列式：`M.det()`

  求秩：`M.rank()`

+ 解线性方程组

$$
\left\{\begin{matrix}
a_{11}x_1 + a_{12}x_2 + ... + a_{1m}x_m = b_1\\
a_{21}x_1 + a_{22}x_2 + ... + a_{2m}x_m = b_2\\
\vdots\\
a_{n1}x_1 + a_{n2}x_2 + ... + a_{nm}x_m = b_n\\
\end{matrix}\right.
$$

矩阵表示：$Ax = B$
$$
A = \begin{bmatrix}
a_{11} & a_{12} & ... & a_{1m}\\
a_{21} & a_{22} & ... & a_{2m}\\
\vdots & \vdots & \ddots & \vdots\\
a_{n1} & a_{n2} & ... & a_{nm}
\end{bmatrix}
\quad 
x = \begin{bmatrix}
x_1\\
x_2\\
\vdots \\
x_m
\end{bmatrix}
\quad 
B = \begin{bmatrix}
b_1\\
b_2\\
\vdots \\
b_n
\end{bmatrix}
$$

```python
x = A.solve_right(B)
```

如果是求解$xA = B$

```python
x = A.solve_left(B)
```

SageMath还集成了右（左）核空间的求解，即$AX = 0$和$XA = 0$的所有解空间

```python
X = A.left_kernel()		# XA = 0
X = A.right_kernel() 	# AX = 0
# 矩阵形式
X = A.right_kernel_matrix()
```

+ 对一般方程（模方程）的求解

```python
# 解一般方程
x,y = var('x,y')
f1 = x + y == 10
f2 = x - y == 0
solve([f1,f2],[x,y])
"""
[[x == 5, y == 5]]
"""

# 解模方程
x,y = var('x,y')
f1 = x + y == 66
f2 = x - y == 23
solve_mod([f1,f2],97)
"""
[(93, 70)]
"""
```

### 5. 多项式环及其函数

+ 多项式环定义

```python
# 定义在有理数环上的一元多项式环
PQ.<x> = PolynomialRing(QQ)
# PQ：多项式环的名字，自定义
# x：变量的名字，自定义

# 以下是等价定义
PQ = PolynomialRing(QQ,'x')
x = PQ.gen()
# 或者
PQ = QQ['t']
```

二元多项式环可以这样定义：

```python
PQ.<x,y> = PolynomialRing(QQ,implementation="generic")
# 等价于
PQ = PolynomialRing(QQ,['x','y'])
x,y = PQ.gens()
```

+ 多项式定义

在定义好多项式环之后`PQ.<x> = PolynomialRing(QQ)`

```python
PQ.<x> = PolynomialRing(QQ)
f = 4*x^3 + 2*x + 1

# 可以用列表实现，列表的元素就是多项式的系数（升幂）
f1 = PQ([1,2,0,4])
print(f1)
"""
4*x^3 + 2*x + 1
"""
```

生成随机多项式

```python
R.<x> = PolynomialRing(Zmod(65537))
degree = 7
S = R.random_element(degree)
print(S)
"""
30809*x^7 + 43675*x^6 + 37650*x^5 + 19133*x^4 + 28302*x^3 + 41523*x^2 + 38027*x + 47569
"""
```

+ 多项式相关运算、函数

多项式分解：

```python
R.<x> = PolynomialRing(Zmod(65537))
degree = 7
f = R.random_element(degree) * R.random_element(degree)
print(f.factor())
"""
(36316) * (x + 11701) * (x + 51377) * (x^6 + 26446*x^5 + 34386*x^4 + 20045*x^3 + 48014*x^2 + 1511*x + 29582) * (x^6 + 28718*x^5 + 5682*x^4 + 36526*x^3 + 24869*x^2 + 16439*x + 26532)
"""
```

多项式最大公因式：

```python
R.<x> = PolynomialRing(Zmod(65537))
degree = 7
g = R.random_element(degree)
f1 = R.random_element(degree) * g
f2 = R.random_element(degree) * g
print(gcd(f1,f2))
"""
x^7 + 17968*x^6 + 5649*x^5 + 15968*x^4 + 39491*x^3 + 10960*x^2 + 11793*x + 43186
"""
```

首一多项式：

```python
R.<x> = PolynomialRing(Zmod(65537))
degree = 7
f = R.random_element()
print(f.monic())
```

多项式的根：

```python
R.<x> = PolynomialRing(Zmod(65537))
degree = 7
f = R.random_element()
print(f.roots())
"""
[(49655, 1), (23052, 1)]
"""
```

Groebner basis：

在已知模方程的一些关系后，我们可以通过Groebner basis得到一些方程的根

```python
G = GF(next_prime(getrandbits(512)))
a = G(getrandbits(512))
b = G(getrandbits(512))
c = a + b + 233
a3 = a^3 
b3 = b^3
c3 = c^3

x,y,z = G['x,y,z'].gens()
I = ideal([z - x - y -233, x^3 - a3, y^3 - b3, z^3 - c3])
B = I.groebner_basis()
print(B)
"""
[x + 2531298875913075551623080047366256605361394651747390176151541916607157452100557045589219380619372113471085086020988642643234366348428408128300737756479339, y + 9415924508923578913613719564884371252781291871935977042763901912456038422181722504308404500554549074832959153436325409624718881868450958162294882284278023, z + 1071131270017849170544175220642133725088124729933657075977649390709780255972543853576917359530421767134795718578905900102821973462764119981798880043620730]
"""
```

分别是a,b,c的值

### 6. 格相关函数

常用的两个：

+ LLL

```python
M = Matrix.random(ZZ,5,5)
M.LLL()
```

+ CopperSmith算法