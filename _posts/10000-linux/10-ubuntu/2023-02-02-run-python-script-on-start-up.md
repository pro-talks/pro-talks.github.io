---
layout: post
title: "Linux - Run python script on start up"
date: 2023-02-02 12:00:00 +0300
categories: linux script
permalink: linux/run-python-script-on-start-up
---

# Linux - How to run a script on startup
1. Wrap your python script in bash script `start.sh`
```bash
source /home/serzh/.../.env
python3.10 /home/serzh/.../bot.py
```

2. Create service
```bash
sudo nano /etc/systemd/system/myscript.service
```

3. Paste the following into file
  ```bash
   Description=My script service description
   
   Wants=network.target
   After=syslog.target network-online.target
   
   [Service]
   Type=simple
   ExecStart=sudo -u serzh /usr/bin/bash /home/serzh/.../start.sh
   Restart=on-failure
   RestartSec=10
   KillMode=mixed
   
   [Install]
   WantedBy=multi-user.target
  ```

3. Reload services
```bash
sudo systemctl daemon-reload 
```

4. Enable the service
```bash
sudo systemctl enable myscript.service
```

5. Start the service
```bash
sudo systemctl start myscript.service
```

6. Check the status of your service
```bash
sudo systemctl status myscript.service
```

7. Restart service
```bash
sudo systemctl restart myscript.service
```

## How to check the logs of service?
```bash
journalctl -u myscript.service
```


## How to log into journalctl for a python program?
```python
import logging
import sys

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# this is just to make the output look nice
formatter = logging.Formatter(
    fmt="%(asctime)s %(name)s.%(levelname)s: %(message)s", datefmt="%Y.%m.%d %H:%M:%S")

# this logs to stdout and I think it is flushed immediately
handler = logging.StreamHandler(stream=sys.stdout)
handler.setFormatter(formatter)
logger.addHandler(handler)

logger.info("Hello!")
```

