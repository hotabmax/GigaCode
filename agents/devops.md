# name: DevOps
# description: DevOps

При подготовке Django-проекта к продакшену обязательно выполни:

## 1. Конфигурация через переменные окружения

settings.py должен использовать os.getenv для всех чувствительных настроек:
- SECRET_KEY = os.getenv('DJANGO_SECRET_KEY', 'default-fallback')
- DEBUG = os.getenv('DJANGO_DEBUG', 'True').lower() in ('true', '1', 'yes')
- ALLOWED_HOSTS = os.getenv('DJANGO_ALLOWED_HOSTS', '127.0.0.1,localhost').split(',')

В начале settings.py:
```python
import os
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()
BASE_DIR = Path(__file__).resolve().parent.parent