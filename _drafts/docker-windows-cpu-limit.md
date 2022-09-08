---
layout: post
title:  "windows + linux + docker container + no docker 에서 공통으로 사용할수 있는 cpu 개수 구하기"
date:   2022-09-02 11:29:57 +0900
categories: docker
tags: [numpy, scipy]
---




container 에 할당된 cpu 개수는 다음 코드처럼 파일을 읽어서 확인 할 수 있다.

```python
import os
import sys
import math
import platform

def get_cpu_count():    
    print("platform.system() = ",platform.system()) # Darwin,Linux,Windows
    print("sys.platform      = ",sys.platform) # win32, linux , darwin
    quota_path ="/sys/fs/cgroup/cpu/cpu.cfs_quota_us" # default -1
    period_path="/sys/fs/cgroup/cpu/cpu.cfs_period_us"
    if os.path.exists(quota_path) and os.path.exists(period_path):
        # linux 
        with open(quota_path) as fp:
            cfs_quota_us = int(fp.read())
        with open(period_path) as fp:
            cfs_period_us = int(fp.read())

        if cfs_quota_us < 0: # -1 : unlimited
            return os.cpu_count()

        cpus = math.floor(cfs_quota_us / cfs_period_us)
        if cpus < 1: 
            return 1  # 1 미만 이면 그냥 cpu 1 개로 리턴.
        else:
            return cpus
    else:
        return os.cpu_count()

# os.environ['OMP_NUM_THREADS']= get_cpu_count()
```
