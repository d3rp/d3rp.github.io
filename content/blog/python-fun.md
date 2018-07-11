---
title: "Python Fun"
date: 2018-02-04T22:05:58+02:00
draft: true
tags: ["Development", "Python", "fast", "Blogging"]
categories: ["Development"]
series: ["Python Web Dev"]
project_url: "https://github.com/rojun/pronounce"
---

Some fun on Python
------------------

{{< highlight python "linenos=table,hl_lines=8 15-17,linenostart=199" >}}

import io
import os
from datetime import datetime
from multiprocessing import Lock, Process

LOG_FILE = "times.log"
lock = Lock()

def time_this_locked(func):
    def wrapper(*args, **kwargs):
        global lock
        start = datetime.now()
        res = func(*args, **kwargs)
        elapsed = datetime.now() - start
        lock.acquire()
        try:
            with open(LOG_FILE, "a") as out_file:
                out_file.write(str(elapsed) + "\n")
        finally:
            lock.release()
        return res

    return wrapper


def gen(MAX):
    i = 0
    while i<MAX:
        i+=1
        yield i


@time_this_locked
def iterate():
    for _ in gen(10000):
        for _ in gen(1000):
            2 ** 4


def f(x):
    return x*x


def main():
    if os.path.exists(LOG_FILE):
        os.remove(LOG_FILE)

    for i in range(1,11):
        Process(target=iterate).start()
    

if __name__ == '__main__':
    main()

{{< / highlight >}}

