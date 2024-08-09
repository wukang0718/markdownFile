# js中文转Base64

```javascript
function utf8ToBase64(str) {
  const encoder = new TextEncoder();
  const data = encoder.encode(str);
  const base64Encoded = btoa(String.fromCharCode(...new Uint8Array(data.buffer)));
  return base64Encoded;
}
```