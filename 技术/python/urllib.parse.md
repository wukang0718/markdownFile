# python解析url中的文件名

```python
import os
from urllib.parse import urlparse, unquote


def get_file_name(url):
    parsed_url = urlparse(url)
    file_name = os.path.basename(parsed_url.path)
    # 做urldecode
    decode_file_name = unquote(file_name)
    return decode_file_name
```