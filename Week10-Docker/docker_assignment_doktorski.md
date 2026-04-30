## Task 1.1 — Change the Python version

Changed `FROM python:3.11-slim` to `FROM python:3.9-slim` in the Dockerfile, then rebuilt and ran.

```
$ docker build -t hello-docker .
#0 building with "orbstack" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 181B done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/python:3.9-slim
#2 DONE 1.6s

#3 [internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [internal] load build context
#4 transferring context: 181B done
#4 DONE 0.0s

#5 [1/4] FROM docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338d3060f261f53f144965f755599aab1acda1e13cf1731b1b
#5 resolve docker.io/library/python:3.9-slim@sha256:... done
#5 ...downloading layers...
#5 DONE 2.3s

#6 [2/4] WORKDIR /app
#6 DONE 0.9s

#7 [3/4] RUN pip install numpy==1.26.4
#7 2.576 Collecting numpy==1.26.4
#7 2.771   Downloading numpy-1.26.4-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (14.2 MB)
#7 3.418      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 14.2/14.2 MB 30.8 MB/s eta 0:00:00
#7 3.470 Installing collected packages: numpy
#7 4.607 Successfully installed numpy-1.26.4
#7 DONE 5.3s

#8 [4/4] COPY hello.py .
#8 DONE 0.0s

#9 exporting to image
#9 exporting layers 0.2s done
#9 writing image sha256:437e9ff1fe6720e85743c3644373125b14ca8a639db4ee02422ba4170fae1495 done
#9 naming to docker.io/library/hello-docker done
#9 DONE 0.2s

$ docker run --rm hello-docker
Hello from Python 3.9 inside a container!
```

---

## Task 1.2 — Break and fix the Dockerfile

### Step 1: Replace hello.py and rebuild — failed run

```
$ docker build -t hello-docker .
#0 building with "orbstack" instance using docker driver
...
#7 [3/4] RUN pip install numpy==1.26.4
#7 CACHED
#8 [4/4] COPY hello.py .
#8 DONE 0.0s
#9 exporting to image
#9 ...done
#9 naming to docker.io/library/hello-docker done
#9 DONE 0.0s

$ docker run --rm hello-docker
Traceback (most recent call last):
  File "/app/hello.py", line 1, in <module>
    import sys, pandas
ModuleNotFoundError: No module named 'pandas'
```

### Step 2: Fix — add pinned pandas, rebuild and run successfully

Added `RUN pip install pandas==2.2.2` to the Dockerfile (replaced the numpy line)

```
$ docker build -t hello-docker .
#0 building with "orbstack" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 181B done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/python:3.9-slim
#2 DONE 0.3s

#3 [internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.1s

#4 [1/4] FROM docker.io/library/python:3.9-slim@sha256:2d97f6910b...
#4 DONE 0.0s

#5 [2/4] WORKDIR /app
#5 CACHED

#6 [internal] load build context
#6 transferring context: 66B done
#6 DONE 0.0s

#7 [3/4] RUN pip install pandas==2.2.2
#7 2.263 Collecting pandas==2.2.2
#7 2.454   Downloading pandas-2.2.2-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (15.7 MB)
#7 2.946      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 15.7/15.7 MB 52.2 MB/s eta 0:00:00
#7 3.054 Collecting python-dateutil>=2.8.2
#7 3.105      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 229.9/229.9 kB 40.6 MB/s eta 0:00:00
#7 3.192 Collecting pytz>=2020.1
#7 3.246      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 510.5/510.5 kB 65.9 MB/s eta 0:00:00
#7 3.594 Collecting numpy>=1.22.4
#7 3.877      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 13.9/13.9 MB 49.7 MB/s eta 0:00:00
#7 3.947 Collecting tzdata>=2022.7
#7 4.004      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 349.3/349.3 kB 31.9 MB/s eta 0:00:00
#7 4.068 Collecting six>=1.5
#7 4.205 Installing collected packages: pytz, tzdata, six, numpy, python-dateutil, pandas
#7 7.054 Successfully installed numpy-2.0.2 pandas-2.2.2 python-dateutil-2.9.0.post0 pytz-2026.1.post1 six-1.17.0 tzdata-2026.2
#7 DONE 8.1s

#8 [4/4] COPY hello.py .
#8 DONE 0.0s

#9 exporting to image
#9 exporting layers 0.4s done
#9 writing image sha256:12bb6b56818b805efbc1570555dded523648e6098702d5f6930b458e06d81b27 done
#9 naming to docker.io/library/hello-docker done
#9 DONE 0.4s

$ docker run --rm hello-docker
Python 3.9, pandas 2.2.2
```

### Final Dockerfile

```dockerfile
FROM python:3.9-slim
WORKDIR /app
RUN pip install pandas==2.2.2
COPY hello.py .
CMD ["python", "hello.py"]
```

---

## Question 2.1 — Why pin?

Writing `RUN pip install pandas` without a version means Docker will fetch whatever is the latest pandas release at build time. This is a reproducibility problem because the version will change over time, if someone rebuilds the image for example a few months later, they can get a different pandas version that includes breaking changes compared to the version that was tested before.

## Question 2.2 — Recipe or cake?

Sending the `Dockerfile` and source files is better for reproducible research than sending the prebuilt image. Dockerfile is a human readable text file that makes every build decision explicit and everyone can inspect exactly which base image, package versions and commands were used, and can reason about security vulnerabilities or other issues. A binary image is opaque, it cannot be diffed or audited for what is inside.
