# js下载svg图片

```javascript
function downloadByUrl(url, fileName) {
  const a = document.createElement('a');
  a.download = fileName;
  a.href = url;
  a.click();
}

function utf8ToBase64(str) {
  const encoder = new TextEncoder();
  const data = encoder.encode(str);
  const base64Encoded = btoa(String.fromCharCode(...new Uint8Array(data.buffer)));
  return base64Encoded;
}

function downloadSvg(svg, fileName) {
  const svgCode = new XMLSerializer().serializeToString(svg);
  const svgDataUrl = `data:image/svg+xml;base64,${utf8ToBase64(svgCode)}`;
  downloadByUrl(svgDataUrl, fileName);
}
```