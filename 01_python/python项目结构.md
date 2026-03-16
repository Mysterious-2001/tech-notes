# 包导入

```python
# standard library
import os
import sys
import logging

# third party
import django
import requests

# internal modules
from project.service.doc import DocHandler
from project.utils.logger import logger
```

不要用 `import *`  
避免循环依赖  
import 尽量放在文件顶部  


