# THREE.GLTFLoader: Unknown extension "KHR_materials_pbrSpecularGlossiness

three.js的较新版本不包含此模型要求的`spec/gloss PBR material`。因此，这里没有快速的解决方法，必须将模型转换为金属/粗糙的PBR材料。这可能很慢 (必须重写纹理)

两种转换方式： 运行时转换、离线转换，建议使用离线转换

1. 运行时转换

```javascript
import { WebIO } from '@gltf-transform/core';
import { KHRONOS_EXTENSIONS } from '@gltf-transform/extensions';
import { metalRough } from '@gltf-transform/functions';

// Load model in glTF Transform.
const io = new WebIO().registerExtensions(KHRONOS_EXTENSIONS);
const document = await io.read('path/to/model.glb');

// Convert materials.
await document.transform(metalRough());

// Write back to GLB.
const glb = await io.writeBinary(document);

// Load model in three.js.
const loader = new GLTFLoader();
loader.parse(glb.buffer, '', (gltf) => {
  scene.add(gltf.scene);
  // ...
});
```

1. 离线转换

```bash
npm install --global @gltf-transform/cli

gltf-transform metalrough input.glb output.glb
```

### 参考文档

https://stackoverflow.com/questions/75886578/three-gltfloader-unknown-extension-khr-materials-pbrspecularglossiness

