# Django NATS

[![GitHub](https://img.shields.io/github/license/C0D1UM/django-nats-client)](https://github.com/C0D1UM/django-nats-client/blob/main/LICENSE)
[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/C0D1UM/django-nats-client/CI)](https://github.com/C0D1UM/django-nats-client/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/C0D1UM/django-nats-client/branch/main/graph/badge.svg?token=PN19DJ3SDF)](https://codecov.io/gh/C0D1UM/django-nats-client)
[![PyPI](https://img.shields.io/pypi/v/django-nats-client)](https://pypi.org/project/django-nats-client/)  
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/django-nats-client)](https://github.com/C0D1UM/django-nats-client)
[![Django Version](https://img.shields.io/badge/django-3.1%20%7C%203.2%20%7C%204.0%20%7C%204.1-blue)](https://github.com/C0D1UM/django-nats-client)

## Features

- Wrapper of NATS's [nats-py](https://github.com/nats-io/nats.py)
- Django management command to listen for incoming NATS messages
- Automatically serialize/deserialize message from/to JSON format
- Easy-to-call method for sending NATS messages

## Installation

```bash
pip install django-nats-client
```

## Setup

1. Add `nats_client` into `INSTALLED_APPS`

   ```python
   # settings.py

   INSTALLED_APPS = [
       ...
       'nats_client',
   ]
   ```

1. Put NATS connection configuration in settings

   ```python
   # settings.py

   NATS_OPTIONS = {
       'servers': ['nats://localhost:4222'],
       'max_reconnect_attempts': 2,
       'connect_timeout': 1,
       ...
   }
   NATS_DEFAULT_SUBJECT = 'default'
   ```

## Usage

### Listen for messages

1. Create a new callback method and register

   ```python
   # common/nats.py

   import nats_client

   @nats_client.register
   def get_year_from_date(date: str):
       return date.year

   # custom subject
   @nats_client.register('subject')
   def current_time():
       return datetime.datetime.now().strftime('%H:%M')

   # custom method name
   @nats_client.register('subject', 'get_current_time')
   def current_time():
       return datetime.datetime.now().strftime('%H:%M')
   ```

1. Import previously file in `ready` method of your `apps.py`

   ```python
   # common/apps.py

   class CommonConfig(AppConfig):
       ...

       def ready(self):
           import common.nats
   ```

1. Run listener management command

   ```bash
   python manage.py nats_listener
   ```

### Sending message

```python
import nats_client

arg = 'some arg'
nats_client.send(
    'subject_name',
    'method_name',
    arg,
    keyword_arg=1,
    another_keyword_arg=2,
)
```

Examples

```python
import nats_client

year = nats_client.send('default', 'get_year_from_date', datetime.date(2022, 1, 1))  # 2022
current_time = nats_client.send('default', 'get_current_time')  # 12:11
```

## Settings

| Key                    | Required | Default   | Description                                       |
|------------------------|----------|-----------|---------------------------------------------------|
| `NATS_OPTIONS`         | Yes      |           | Configuration to be passed in `nats.connect()`    |
| `NATS_DEFAULT_SUBJECT` | No       | 'default' | Default subject for registering callback function |

## Development

### Requirements

- Docker
- Python
- Poetry

### Linting

```bash
make lint
```

### Testing

```bash
make test
```

### Fix Formatting

```bash
make yapf
```
