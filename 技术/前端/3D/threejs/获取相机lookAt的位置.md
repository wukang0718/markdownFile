# 获取相机lookAt的位置

可以通过将相机的位置和朝向信息计算出来的视图矩阵来得到。

具体来说，首先可以使用 `getWorldDirection` 方法来获取相机的朝向向量。然后，通过将相机的位置加上该向量的一定倍数，就可以得到相机正在看向的点。这里的倍数可以根据具体情况自行调整，以保证获取的点在相机的视锥体内。

```javascript
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
const distance = 1000;  // 倍数
const target = camera.position.clone()
	.add(camera.getWorldDirection(new THREE.Vector3()).multiplyScalar(distance));
```

在上面的代码中，我们首先通过 `getWorldDirection` 方法获取相机的朝向向量 `direction`。然后，通过将相机位置加上该向量的 `distance` 倍，就可以得到相机正在看向的点 `target`