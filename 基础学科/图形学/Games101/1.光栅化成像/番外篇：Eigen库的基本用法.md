# 向量

```
Eigen::Vector3f vec{1.0f, 2.0f, 3.0f}; // 创建
float x = vec.x(); // 获取元素
float y = vec.y();
float z = vec.z();
vec << 1.0f, 2.0f, 3.0f; // 设置元素
vec = vec1.cwiseProduct(vec2); // 对应位置相乘
float value = vec1.dot(vec2); // 点积
vec = vec1.croww(vec2); // 叉积
float* value_pointer = vec.data(); // 获取数据的指针
```

# 矩阵

```
mat.col(0) = vec; // 第一行赋值
mat.row(0) = vec; // 第一列赋值
```
