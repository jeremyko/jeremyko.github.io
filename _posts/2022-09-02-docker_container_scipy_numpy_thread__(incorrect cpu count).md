---
layout: post
title: '(python) docker container 에서 scipy, numpy 사용 시 thread 과다 생성 문제 (incorrect cpu count)'
date: 2022-09-02 11:29:57 +0900
# categories: python # 별도 폴더로 관리된다 !!!
tags: [python, numpy, scipy]
---

개발 중인 python 프로젝트에서 scipy 패키지 interpolate 를 사용하는 코드가 있는데, docker 기반 클라우드 환경에서 실행하니 먹통이 되고 멈추는 현상이 발생 했다. 하지만, 이 코드를 내 랩탑 에서 돌려보면, 정상 동작한다.

문제의 원인을 찾아보니, docker container 를 생성하면서 할당한 cpu 개수와 실제 머신의 전체 CPU 개수 불일치로 인한 문제였다. 이 현상에 대해 정리 해 본다.

-   <span style="color:{{site.span_h4_color}}">docker container 기반에서 [OpenBLAS](https://github.com/xianyi/OpenBLAS), [MLK](https://en.wikipedia.org/wiki/Math_Kernel_Library) 를 사용하는 numpy, scipy 등의 패키지 내부에서 과도한 thread 생성이 안되게 주의 해야 한다.</span>

-   <span style="color:{{site.span_h4_color}}">비록 명시적으로 thread 를 사용하지 않았어도, numpy, scipy 내부적으로는 multi thread 가 생성 된다는 것을 알아둘 필요가 있다.</span>

좀 더 자세히 알아보면,

<!-- ### 머신 전체 cpu 를 사용 가능하다고 잘못 판단, 과도한 thread 를 생성 -->
<h3> <span style="color:{{site.span_h3_color}}"> 머신 전체 cpu 를 사용 가능하다고 잘못 판단, 과도한 thread 를 생성 </span> </h3>

내가 만든 python 코드에는 thread 를 사용 한 적도 없는데 왜 thread 가 문제 인가 할 수도 있겠지만, numpy, scipy 는 내부적으로 OpenBLAS 혹은 MLK 라는 라이브러리를 사용하고 있고, multi thread를 활용 하고 있다.

사용 중인 numpy, scipy 가 어떤 라이브러리를 사용 중인지는 다음 명령으로 확인 할 수 있다.

    python -c "import numpy; numpy.show_config()"

    python -c "import scipy; scipy.show_config()"

pip 를 사용해서 numpy, scipy 등을 설치한 경우, windows 에서는 MLK, 리눅스(Ubuntu)에서는 OpenBLAS 를 사용하는 패키지가 설치된다.

그래서 이렇게 내부적으로 thread 가 생성이 되는데, 특별한 조치가 없다면 머신 의 모든 cpu 개수 만큼 thread 를 생성하게 된다.

여기서 문제가 발생 하는데, docker container 생성 시 할당한 cpu 개수를 OpenBLAS, MLK 에서 정확히 모른다는 것이다 (incorrect cpu count).

docker container 는 가상 머신 이 아니기 때문에, docker container 에서 `/proc/cpuinfo` 를 열어보면, 실제 머신 이 가진 모든 cpu 가 출력 된다.

즉 OpenBLAS, MLK 입장에서는 머신 의 모든 cpu 를 사용 할 수 있다 라고 잘못 판단을 하게 되고, 결과적으로 cpu 개수 만큼 worker thread 들을 생성하게 되는 것이다.

실제로 프로젝트에서는 docker container 생성 시 할당된 cpu 개수는 4 개였고 , 머신 이 가진 cpu는 48 개였다. 문제가 되는 코드 실행 시, 전체 thread 개수가 수 십 개가 생성이 되면서, 할당된 cpu 4 개로는 감당 못할 수준으로 느려졌고... 멈추는 현상이 나왔다.

<!-- ### 환경변수 설정으로 과도한 thread 생성을 제어 -->
<h3> <span style="color:{{site.span_h3_color}}"> 환경변수 설정으로 과도한 thread 생성을 제어 </span> </h3>

그래서 이 문제를 해결하기 위해서는 thread 가 마구 생성되는 것을 제어할 필요가 있다. 다음처럼 환경 변수에 원하는 thread 개수를 설정 하면 된다.

그런데 OpenBLAS, MLK 는 둘 중에 하나만 사용 되므로, 현재 설치된 라이브러리가 OpenBLAS 인지 MLK 인지 확인 후 그에 따라 필요한 환경 변수만 설정 해주고 python 코드를 실행하면 된다.

    직접 shell 에서 설정 : OpenBLAS 인 경우
    export OPENBLAS_NUM_THREADS=4

    직접 shell 에서 설정 : MLK 인 경우
    export OMP_NUM_THREADS=4

코드 안에서 처리 하려면, 다음 코드를 최 상위 부분에 추가 하면 된다. (**주의 : import numpy 혹은 import scipy 혹은 import pandas 보다 먼저 해주지 않으면 작동 하지 않는다**)

```python
import os

# numpy,scipy,pandas import 보다 먼저 해줘야지만 동작 함
# OpenBLAS 인 경우
os.environ['OPENBLAS_NUM_THREADS']= "4"

# MLK 인 경우
os.environ['OMP_NUM_THREADS']= "4"

import numpy as np
import scipy
import pandas as pd
```

참고로 OMP_NUM_THREADS 환경 변수는 OpenBLAS, MLK 모두에게 가능하다. 그러므로 `OMP_NUM_THREADS` 하나만 사용 해도 무방하다. 환경 변수의 우선 순위는 **OPENBLAS_NUM_THREADS > OMP_NUM_THREADS** 순서 이다.

container 에 할당된 cpu 개수는 다음 코드처럼 파일을 읽어서 확인 할 수 있다.

```python
import os
import sys
import math
import platform

def get_cpu_count():
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

# os.environ['OMP_NUM_THREADS']= str(get_cpu_count())
```

<!-- ### 작성한 코드가 이미 thread 를 사용 하는지 여부 -->
<h3> <span style="color:{{site.span_h3_color}}"> 작성한 코드가 이미 멀티 thread 로 동작되는지 여부 </span> </h3>

한 가지 더 고려해 봐야 할 것이 있는데, 작성한 코드가 이미 multi thread를 사용하고 있는 경우이다. 이 경우엔 OpenBLAS, MLK 이 만드는 thread와 충돌이 발생되어 deadlocks 등의 문제가 발생될 수 있다고 한다 (왜 이런식으로 동작되는지는 파악을 못했다. 다만 문제가 발생한다는 글들을 많이 검색할수 있었으므로 이에 따르기로 한다).

이를 방지하기 위해서는 OpenBLAS, MLK 의 thread 개수를 1개로, 오히려 낮춰야 한다.

    OpenBLAS 인 경우
    export OPENBLAS_NUM_THREADS=1
    os.environ['OPENBLAS_NUM_THREADS']= "1"

    MLK 인 경우
    export OMP_NUM_THREADS=1
    os.environ['OMP_NUM_THREADS']= "1"

<!-- ### 코드 동작 환경을 먼저 분석하고 진짜 thread 문제 인지 파악 필요 -->
<h3> <span style="color:{{site.span_h3_color}}"> 코드 동작 환경을 먼저 분석하고 진짜 thread 문제 인지 파악 필요 </span> </h3>

개발자라면 당연하게 여기겠지만, 자신의 코드가 동작하는 환경을 먼저 분석하고, 문제점이 이에 해당되는 경우에만 thread 개수 설정을 해야 한다. 임의로 개수 조정하는 것은 오히려 성능을 더 저하 시키는 요인이 될 수도 있다는 것을 유념해 둘 필요가 있다.

<h3> <span style="color:{{site.span_h3_color}}"> 참고 </span> </h3>
{: .notice--accent}

[cgroupfs 참고](https://tech.kakao.com/2020/06/29/cgroup-driver/)  
[docker runtime option](https://docs.docker.com/config/containers/resource_constraints/#configure-the-default-cfs-scheduler)  
[Completely Fair Scheduler Bandwidth Control](https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt)  
[cgroups — Linux manual page](https://man7.org/linux/man-pages/man7/cgroups.7.html)  
[https://github.com/obspy/obspy/wiki/Notes-on-Parallel-Processing-with-Python-and-ObsPy](https://github.com/obspy/obspy/wiki/Notes-on-Parallel-Processing-with-Python-and-ObsPy)  
[https://pipelines.lsst.io/v/foo/modules/lsst.pipe.base/command-line-task-parallel-howto.html](https://pipelines.lsst.io/v/foo/modules/lsst.pipe.base/command-line-task-parallel-howto.html)  
[https://stackoverflow.com/questions/16047306/how-is-docker-different-from-a-virtual-machine?rq=1](https://stackoverflow.com/questions/16047306/how-is-docker-different-from-a-virtual-machine?rq=1)  
[https://numpy.org/install](https://numpy.org/install)  
[https://www.slideshare.net/JoongiKim/soscon-2017-backendai](https://www.slideshare.net/JoongiKim/soscon-2017-backendai) 37페이지

### UPDATE

python3.13부터 설정으로 cpu개수를 제한할수 있다고 한다. [https://docs.python.org/3.13/whatsnew/3.13.html#os](https://docs.python.org/3.13/whatsnew/3.13.html#os)

