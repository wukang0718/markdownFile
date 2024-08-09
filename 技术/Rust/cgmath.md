# cgmath

## 使用cgmath

```toml
cgmath = { version = "0.18.0", features = ["mint"] }
```



## 创建Matrix

`new` 方法创建的时候，数据是列优先的

```rust
use cgmath::{Matrix4};

let matrix = Matrix4::new(
    0.08771602494494875,
    0.00027784751108157714,
    -9.943497835969842e-26,
    0.0,
    0.0002778475110814381,
    -0.08771602494490487,
    -8.77164649950054e-8,
    0.0,
    -0.0000015836235895987607,
    0.0004999474918739505,
    -499.949999999749,
    0.0,
    -176.13790344407562,
    175.9172506247769,
    -400.049840057432,
    1.0000000000000002,
);
// 等同于threejs的
const matrix = new Matrix4().fromArray([
    0.08771602494494875,
    0.00027784751108157714,
    -9.943497835969842e-26,
    0,
    0.0002778475110814381,
    -0.08771602494490487,
    -8.77164649950054e-8,
    0,
    -0.0000015836235895987607,
    0.0004999474918739505,
    -499.949999999749,
    0,
    -176.13790344407562,
    175.9172506247769,
    -400.049840057432,
    1.0000000000000002
])
```

### 从矩阵中解析scale

```RUST
fn extract_scale(matrix: &mut Matrix4<f64>) -> Vector3<f64> {
  let mut x = vec3(matrix[0][0], matrix[0][1], matrix[0][2]).magnitude();
  let y = vec3(matrix[1][0], matrix[1][1], matrix[1][2]).magnitude();
  let z = vec3(matrix[2][0], matrix[2][1], matrix[2][2]).magnitude();
  let d = matrix.determinant();
  if d < 0.0 {
      x = -x
  }
  let scale = Vector3 {
      x,
      y,
      z,
  };
  scale
}
```

### 从矩阵中解析translate

```rust
fn extract_translate(matrix: &mut Matrix4<f64>) -> Vector3<f64> {
    let translate = Vector3 {
        x: matrix[3][0],
        y: matrix[3][1],
        z: matrix[3][2],
    };
    translate
}
```

### 矩阵乘法

```rust
let m2 = matrix2 * matrix;
// 等同于 threejs
matrix2.multiply(matrix);
```



## 创建Vector

```rust
use cgmath::{vec4};

let v = vec4(1.0, 1.0, 1.0, 1.0);
```

### 计算向量的长度

```rust
use cgmath::{InnerSpace};

let length = v.magnitude();
// 等同于threejs的
const length = v.length()
```



## 向量乘矩阵

```rust
let result: Vector4 = matrix * v;

// 等同于threejs的
vector.applyMatrix4(matrix)
```