# Eigen的学习

Eigen 的c++ 矩阵的命令

Eigen 是 C++ 中最常用的线性代数库，以下是常用命令总结：

## 头文件与命名空间

cpp

```cpp
#include <Eigen/Dense>
using namespace Eigen;
```

------

## 矩阵类型声明

cpp

```cpp
MatrixXd  m;          // 动态大小，double
MatrixXf  m;          // 动态大小，float
Matrix3d  m;          // 3×3 固定大小，double
Matrix4f  m;          // 4×4 固定大小，float
MatrixXd  m(3, 4);    // 动态 3×4

// 向量
VectorXd  v(5);       // 动态列向量
Vector3d  v;          // 固定 3D 列向量
RowVector3d rv;       // 固定 3D 行向量
```

------

## 初始化

cpp

```cpp
Matrix3d m = Matrix3d::Zero();      // 全零
Matrix3d m = Matrix3d::Ones();      // 全一
Matrix3d m = Matrix3d::Identity();  // 单位矩阵
Matrix3d m = Matrix3d::Random();    // 随机

// 逗号初始化
Matrix2d m;
m << 1, 2,
     3, 4;
```

------

## 基本运算

cpp

```cpp
MatrixXd A, B, C;

C = A + B;         // 加法
C = A - B;         // 减法
C = A * B;         // 矩阵乘法
C = 2.0 * A;       // 标量乘法
C = A.array() * B.array();  // 逐元素乘法（需转 array）
```

------

## 常用操作

cpp

```cpp
A.transpose()       // 转置
A.inverse()         // 逆矩阵
A.determinant()     // 行列式
A.trace()           // 迹
A.norm()            // L2 范数
A.rows()            // 行数
A.cols()            // 列数
A.size()            // 元素总数
A(i, j)             // 访问元素
A.row(i)            // 第 i 行
A.col(j)            // 第 j 列
```

------

## 块操作（子矩阵）

cpp

```cpp
A.block(i, j, p, q)       // 从 (i,j) 起，大小 p×q
A.topLeftCorner(p, q)      // 左上角 p×q
A.bottomRightCorner(p, q)  // 右下角 p×q
A.topRows(p)               // 前 p 行
A.leftCols(q)              // 前 q 列
```

------

## 线性求解

cpp

```cpp
// 求解 Ax = b
VectorXd x = A.lu().solve(b);            // LU 分解
VectorXd x = A.llt().solve(b);           // Cholesky（正定矩阵）
VectorXd x = A.colPivHouseholderQr().solve(b);  // QR 分解（通用）
```

------

## 特征值分解

cpp

```cpp
SelfAdjointEigenSolver<MatrixXd> solver(A);
VectorXd eigenvalues  = solver.eigenvalues();
MatrixXd eigenvectors = solver.eigenvectors();
```

------

## SVD 分解

cpp

```cpp
JacobiSVD<MatrixXd> svd(A, ComputeThinU | ComputeThinV);
MatrixXd U = svd.matrixU();
MatrixXd V = svd.matrixV();
VectorXd S = svd.singularValues();
```

------

## Array vs Matrix

cpp

```cpp
// Array 用于逐元素操作
ArrayXXd arr = A.array();
arr = arr.sqrt();           // 逐元素开方
arr = arr.pow(2);           // 逐元素平方
arr = arr.exp();            // 逐元素 exp

A = arr.matrix();           // 转回 Matrix
```

## 实例

下面逐一详细讲解这四个主题！

------

## 1. 矩阵初始化与类型

cpp

```cpp
#include <Eigen/Dense>
using namespace Eigen;

// ── 固定大小（编译期确定，性能更好）──
Matrix2d  A;          // 2×2 double
Matrix3f  B;          // 3×3 float
Matrix4d  C;          // 4×4 double
Matrix<double, 3, 4> D;   // 自定义 3×4

// ── 动态大小（运行期确定）──
MatrixXd  M(3, 4);    // 3×4 double
MatrixXf  N(5, 5);    // 5×5 float

// ── 向量 ──
Vector3d   v1;               // 固定 3D 列向量
VectorXd   v2(6);            // 动态列向量，长度 6
RowVector3d rv;              // 固定 3D 行向量

// ── 特殊初始化 ──
MatrixXd Z = MatrixXd::Zero(3, 3);       // 全零
MatrixXd O = MatrixXd::Ones(3, 3);       // 全一
MatrixXd I = MatrixXd::Identity(4, 4);   // 单位矩阵
MatrixXd R = MatrixXd::Random(3, 3);     // [-1, 1] 随机
MatrixXd C = MatrixXd::Constant(3, 3, 6.28);  // 全部填充某值

// ── 逗号初始化 ──
Matrix3d m;
m << 1, 2, 3,
     4, 5, 6,
     7, 8, 9;

Vector3d v;
v << 1.0, 2.0, 3.0;

// ── 动态 resize ──
MatrixXd P;
P.resize(4, 4);       // 改变大小（不保留数据）
P.setZero();          // 填零
P.setOnes();          // 填一
P.setIdentity();      // 设为单位矩阵
P.setRandom();        // 随机填充
```

------

## 2. 线性求解 Ax = b

cpp

```cpp
MatrixXd A(3, 3);
VectorXd b(3);

A << 2, 1, -1,
    -3, -1, 2,
    -2,  1, 2;
b << 8, -11, -3;

// ── 方法1：PartialPivLU（方阵，通用）──
VectorXd x1 = A.lu().solve(b);

// ── 方法2：FullPivLU（更稳定，适合病态矩阵）──
VectorXd x2 = A.fullPivLu().solve(b);

// ── 方法3：LLT Cholesky（要求 A 对称正定，最快）──
VectorXd x3 = A.llt().solve(b);

// ── 方法4：LDLT（半正定，比 LLT 更鲁棒）──
VectorXd x4 = A.ldlt().solve(b);

// ── 方法5：QR 分解（超定/欠定方程，最小二乘）──
VectorXd x5 = A.colPivHouseholderQr().solve(b);

// ── 验证残差 ──
double residual = (A * x1 - b).norm();
std::cout << "残差: " << residual << std::endl;

// ── 最小二乘（A 非方阵，如 5×3）──
MatrixXd A2 = MatrixXd::Random(5, 3);
VectorXd b2 = VectorXd::Random(5);
VectorXd x_ls = A2.colPivHouseholderQr().solve(b2);
```

| 方法                    | 适用场景      | 速度 |
| ----------------------- | ------------- | ---- |
| `lu()`                  | 通用方阵      | 快   |
| `llt()`                 | 对称正定      | 最快 |
| `ldlt()`                | 对称半正定    | 快   |
| `colPivHouseholderQr()` | 通用/最小二乘 | 中   |
| `fullPivLu()`           | 病态/奇异矩阵 | 慢   |

------

## 3. 特征值 & SVD 分解

cpp

```cpp
// ══ 特征值分解 ══

// 对称矩阵（推荐，结果为实数）
Matrix3d S;
S << 4, 2, 0,
     2, 3, 0,
     0, 0, 1;

SelfAdjointEigenSolver<Matrix3d> eig(S);
VectorXd eigenvalues  = eig.eigenvalues();    // 升序排列
MatrixXd eigenvectors = eig.eigenvectors();   // 列为特征向量
std::cout << "特征值:\n" << eigenvalues << std::endl;

// 一般矩阵（特征值可能为复数）
MatrixXd G = MatrixXd::Random(3, 3);
EigenSolver<MatrixXd> eig2(G);
auto cvals = eig2.eigenvalues();    // 复数类型
auto cvecs = eig2.eigenvectors();   // 复数特征向量

// ══ SVD 分解  A = U * S * V^T ══
MatrixXd A(4, 3);
A << 1, 0, 0,
     0, 2, 0,
     0, 0, 3,
     1, 1, 1;

JacobiSVD<MatrixXd> svd(A, ComputeThinU | ComputeThinV);
MatrixXd U = svd.matrixU();           // 4×3
MatrixXd V = svd.matrixV();           // 3×3
VectorXd S = svd.singularValues();    // 降序奇异值

std::cout << "U:\n" << U << "\nS:\n" << S << "\nV:\n" << V << std::endl;

// ── 用 SVD 求伪逆 ──
double tol = 1e-9;
VectorXd S_inv = S.array().abs().unaryExpr(
    [tol](double s){ return s > tol ? 1.0/s : 0.0; });
MatrixXd A_pinv = V * S_inv.asDiagonal() * U.transpose();
```

------

## 4. 块操作与切片

cpp

```cpp
MatrixXd M(5, 5);
M << 1,  2,  3,  4,  5,
     6,  7,  8,  9,  10,
     11, 12, 13, 14, 15,
     16, 17, 18, 19, 20,
     21, 22, 23, 24, 25;

// ── 通用块：block(起始行, 起始列, 行数, 列数) ──
MatrixXd B = M.block(1, 1, 3, 3);    // 中间 3×3 子矩阵

// ── 角落块 ──
M.topLeftCorner(2, 2)       // 左上 2×2
M.topRightCorner(2, 2)      // 右上 2×2
M.bottomLeftCorner(2, 2)    // 左下 2×2
M.bottomRightCorner(2, 2)   // 右下 2×2

// ── 行列切片 ──
M.topRows(2)                // 前 2 行
M.bottomRows(2)             // 后 2 行
M.leftCols(2)               // 前 2 列
M.rightCols(2)              // 后 2 列
M.middleRows(1, 3)          // 从第1行起取3行
M.middleCols(1, 3)          // 从第1列起取3列

// ── 单行/列访问 ──
M.row(0)                    // 第 0 行（行向量）
M.col(2)                    // 第 2 列（列向量）

// ── 向量切片 ──
VectorXd v = VectorXd::LinSpaced(10, 0, 9);  // [0,1,...,9]
v.head(3)       // 前 3 个元素
v.tail(3)       // 后 3 个元素
v.segment(2, 4) // 从索引2起取4个元素

// ── 块赋值（原地修改）──
M.block(0, 0, 2, 2) = Matrix2d::Identity();   // 左上角设为单位矩阵
M.col(0).setZero();                            // 第0列清零
M.row(2) = RowVectorXd::Ones(5);              // 第2行设为全一
```

------

以上四个主题涵盖了 Eigen 日常使用的核心内容。

# Eigen 稀疏矩阵 — 有限元入门

------

## 1. 头文件

cpp

```cpp
#include <Eigen/Sparse>
#include <Eigen/SparseCholesky>
#include <Eigen/IterativeLinearSolvers>
using namespace Eigen;

typedef SparseMatrix<double> SpMat;
typedef Triplet<double>      T;
```

------

## 2. 构建稀疏矩阵（三元组法）

有限元中最常用的组装方式：

cpp

```cpp
int n = 100;  // 自由度数量
SpMat K(n, n);  // 刚度矩阵

// ── Step 1: 收集三元组 (row, col, value) ──
std::vector<T> triplets;
triplets.reserve(3 * n);  // 预估非零元数量，提升性能

// 模拟单元刚度矩阵组装（1D 杆单元示例）
for (int e = 0; e < n - 1; e++) {
    int i = e, j = e + 1;
    // 单元刚度贡献
    triplets.push_back(T(i, i,  1.0));
    triplets.push_back(T(i, j, -1.0));
    triplets.push_back(T(j, i, -1.0));
    triplets.push_back(T(j, j,  1.0));
}

// ── Step 2: 一次性填入矩阵 ──
K.setFromTriplets(triplets.begin(), triplets.end());
// 重复位置的值会自动相加（符合有限元叠加原理！）
```

------

## 3. 完整 1D 有限元示例（泊松方程）

求解 `-u'' = f`，两端固定边界

cpp

```cpp
#include <Eigen/Sparse>
#include <Eigen/SparseLU>
#include <iostream>
using namespace Eigen;
typedef SparseMatrix<double> SpMat;
typedef Triplet<double> T;

int main() {
    int n  = 5;               // 内部节点数
    int N  = n + 2;           // 总节点（含边界）
    double h = 1.0 / (n + 1); // 单元长度

    // ── 组装全局刚度矩阵 K ──
    SpMat K(N, N);
    std::vector<T> tri;

    for (int i = 0; i < N; i++) {
        tri.push_back(T(i, i, 2.0 / (h * h)));
        if (i > 0)     tri.push_back(T(i, i-1, -1.0 / (h * h)));
        if (i < N - 1) tri.push_back(T(i, i+1, -1.0 / (h * h)));
    }
    K.setFromTriplets(tri.begin(), tri.end());

    // ── 组装载荷向量 f ──
    VectorXd F = VectorXd::Ones(N);  // f = 1

    // ── 施加边界条件（置1法）──
    // u(0) = 0, u(N-1) = 0
    for (int bc : {0, N-1}) {
        for (SpMat::InnerIterator it(K, bc); it; ++it)
            it.valueRef() = (it.row() == bc) ? 1.0 : 0.0;
        // 同时清零对应行（对称处理）
        K.row(bc) *= 0;
        K.coeffRef(bc, bc) = 1.0;
        F(bc) = 0.0;
    }
    K.makeCompressed();

    // ── 求解 ──
    SparseLU<SpMat> solver;
    solver.analyzePattern(K);
    solver.factorize(K);
    VectorXd u = solver.solve(F);

    std::cout << "位移解 u:\n" << u << std::endl;
    return 0;
}
```

------

## 4. 稀疏矩阵求解器对比

cpp

```cpp
// ── 直接法 ──

// LU 分解（通用非对称）
SparseLU<SpMat> lu_solver;
lu_solver.compute(K);
VectorXd u = lu_solver.solve(F);

// Cholesky（对称正定，FEM 刚度矩阵首选）
SimplicialLLT<SpMat> llt_solver;
llt_solver.compute(K);
VectorXd u = llt_solver.solve(F);

// LDLT（对称半正定，更鲁棒）
SimplicialLDLT<SpMat> ldlt_solver;
ldlt_solver.compute(K);
VectorXd u = ldlt_solver.solve(F);

// ── 迭代法（大规模问题）──

// 共轭梯度法 CG（对称正定）
ConjugateGradient<SpMat, Lower|Upper> cg;
cg.setMaxIterations(1000);
cg.setTolerance(1e-8);
cg.compute(K);
VectorXd u = cg.solve(F);
std::cout << "迭代次数: " << cg.iterations() << std::endl;
std::cout << "残差:     " << cg.error()      << std::endl;

// BiCGSTAB（非对称矩阵）
BiCGSTAB<SpMat> bicg;
bicg.compute(K);
VectorXd u = bicg.solve(F);
```

| 求解器              | 适用矩阵   | 规模   | 推荐场景   |
| ------------------- | ---------- | ------ | ---------- |
| `SimplicialLLT`     | 对称正定   | 中小   | 结构静力学 |
| `SimplicialLDLT`    | 对称半正定 | 中小   | 含约束问题 |
| `SparseLU`          | 通用方阵   | 中     | 非对称问题 |
| `ConjugateGradient` | 对称正定   | 大规模 | 大型结构   |
| `BiCGSTAB`          | 非对称     | 大规模 | 流体/热力  |

------

## 5. 稀疏矩阵常用操作

cpp

```cpp
SpMat K(n, n);

// ── 基本信息 ──
K.rows()          // 行数
K.cols()          // 列数
K.nonZeros()      // 非零元数量
K.isCompressed()  // 是否压缩存储

// ── 压缩与优化 ──
K.makeCompressed();      // CSC 压缩格式（求解前必须调用）
K.uncompress();          // 取消压缩
K.reserve(n * 7);        // 预留非零元空间

// ── 访问元素（慢，避免在循环中使用）──
double v = K.coeff(i, j);
K.coeffRef(i, j) += val;   // 修改元素

// ── 遍历非零元 ──
for (int col = 0; col < K.outerSize(); col++) {
    for (SpMat::InnerIterator it(K, col); it; ++it) {
        std::cout << "(" << it.row() << "," 
                  << it.col() << ") = " 
                  << it.value() << "\n";
    }
}

// ── 矩阵运算 ──
SpMat A = K + K;           // 加法
SpMat B = K * K;           // 矩阵乘法
VectorXd y = K * x;        // 矩阵向量乘（核心操作）
SpMat Kt = K.transpose();  // 转置
```

------

## 6. 有限元边界条件处理技巧

cpp

~~~cpp
// ── 方法1：置1法（简单快速）──
void applyBC_penalty(SpMat& K, VectorXd& F, int dof) {
    K.row(dof) *= 0;
    K.col(dof) *= 0;
    K.coeffRef(dof, dof) = 1.0;
    F(dof) = 0.0;
}

// ── 方法2：罚函数法 ──
double penalty = 1e20;
K.coeffRef(dof, dof) += penalty;
F(dof) += penalty * prescribed_value;

// ── 方法3：缩减自由度法（效率最高）──
// 直接删除约束自由度对应的行列
// 需要建立 free_dofs 索引映射
std::vector<int> free_dofs;
for (int i = 0; i < n; i++)
    if (!is_constrained[i]) free_dofs.push_back(i);
```

---

## 典型 FEM 工作流程
```
建立网格节点/单元
       ↓
循环单元 → 计算单元刚度矩阵 ke
       ↓
用 Triplet 组装全局 K 和 F
       ↓
施加边界条件
       ↓
K.makeCompressed()
       ↓
选择求解器 → 求解 u = K⁻¹F
       ↓
后处理（应力/应变计算）
~~~

------

**有限元自己写代码时，Eigen 里最常用到的操作清单**

------

## 有限元编程 Eigen 速查表

### 单元计算阶段

cpp

```cpp
// 节点坐标
Vector2d p1, p2, p3;

// 小矩阵运算（单元刚度 ke）
Matrix<double,6,6> ke;
Matrix<double,3,6> B;
Matrix<double,3,3> D;
ke = t * A * B.transpose() * D * B;

// 行列式、逆（Jacobian）
Matrix2d J;
double detJ = J.determinant();
Matrix2d Jinv = J.inverse();
```

### 组装阶段

cpp

```cpp
// Triplet 组装
std::vector<Triplet<double>> triplets;
triplets.push_back(T(global_i, global_j, ke(i,j)));
K.setFromTriplets(triplets.begin(), triplets.end());

// 自由度映射
std::vector<int> dof = {2*n, 2*n+1, ...};  // 节点→自由度
```

### 边界条件

cpp

```cpp
K.coeffRef(dof, dof) = 1e20;   // 罚函数法
F(dof) = 0.0;                  // 置零法
```

### 求解阶段

cpp

```cpp
SimplicialLDLT<SpMat> solver;
solver.compute(K);
VectorXd u = solver.solve(F);
```

### 后处理

cpp

```cpp
// 取单元节点位移
VectorXd ue = u(dofs);          // 切片
VectorXd stress = D * B * ue;   // 应力计算
```

### 动力学附加

cpp

```cpp
// 广义特征值 Kφ = λMφ
GeneralizedSelfAdjointEigenSolver<MatrixXd> ges(K_dense, M_dense);
VectorXd freq = ges.eigenvalues().cwiseSqrt();
```

# 简单的命令

## `Constant()` 详解

### 基本用法

cpp

```cpp
// 静态（固定大小）
Matrix3d A = Matrix3d::Constant(3.14);       // 3×3 全填 3.14

// 动态（需指定大小）
MatrixXd B = MatrixXd::Constant(3, 4, 2.0); // 3×4 全填 2.0
VectorXd v = VectorXd::Constant(5, 1.0);    // 长度5 全填 1.0
```

------

### 有限元中常见用途

cpp

```cpp
// 初始化载荷向量
VectorXd F = VectorXd::Constant(n_dof, 0.0);  // 等价于 setZero()

// 均布体力
VectorXd fb = VectorXd::Constant(2, rho * g); // [ρg, ρg]

// 初始温度场
VectorXd T0 = VectorXd::Constant(n_nodes, 20.0); // 全场20°C
```

------

### 与类似命令对比

cpp

```cpp
VectorXd::Constant(n, val)  // 全部 = val
VectorXd::Zero(n)           // 全部 = 0
VectorXd::Ones(n)           // 全部 = 1
VectorXd::LinSpaced(n, a, b)// 从a到b均匀分布
```

------

### 原地版本

cpp

```cpp
MatrixXd A(3,3);
A.setConstant(5.0);   // 等价于 A = MatrixXd::Constant(3,3,5.0)
```

