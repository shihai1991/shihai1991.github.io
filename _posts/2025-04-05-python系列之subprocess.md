---
published: true
layout: post
title: python系列-subprocess
category: python
tags:
  - python
time: '2025.04.05 15:43:00'
---

# subprocess模块和multiprocess模块
## subprocess模块
`subprocess`模块允许我们`spawn`一个新的进程。
```python
>>> subprocess.run(["ls", "-l"])  # doesn't capture output
CompletedProcess(args=['ls', '-l'], returncode=0)

>>> subprocess.run("exit 1", shell=True)
CompletedProcess(args=['ls', '-l'], returncode=0)
```

如果我们要对创建出来的进程结果进行判断，那我们就需要添加一个`check=True`的参数。
```python
>>> subprocess.run("exit 1", shell=True, check=True)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib64/python3.9/subprocess.py", line 528, in run
    raise CalledProcessError(retcode, process.args,
subprocess.CalledProcessError: Command 'exit 1' returned non-zero exit status 1.
```
当然，我们使用`subprocess.check_x()`函数也都会对返回码进行检查，如：`subprocess.check_call()`、`subprocess.check_output()`。  
实际上，我们在日常中使用频率最高的`subprocess.getstatusoutput()`函数实际调用的执行函数是`subprocess.checkout_output()`，而`subprocess.checkout_output()`调用的执行函数则是`subprocess.run(check=True)`。  
这里针对正常的情况是这样的，但是如果某些进程已经消亡无法跟踪其状态，你在`subprocess.check_x()`函数时可能就得到不正确的执行结果。
```
        def _internal_poll(self, _deadstate=None, _del_safe=_del_safe):
            """Check if child process has terminated.  Returns returncode
            attribute.

            This method is called by __del__, so it cannot reference anything
            outside of the local scope (nor can any methods it calls).

            """
            if self.returncode is None:
                if not self._waitpid_lock.acquire(False):
                    # Something else is busy calling waitpid.  Don't allow two
                    # at once.  We know nothing yet.
                    return None
                try:
                    if self.returncode is not None:
                        return self.returncode  # Another thread waited.
                    pid, sts = _del_safe.waitpid(self.pid, _del_safe.WNOHANG)
                    if pid == self.pid:
                        self._handle_exitstatus(sts)
                except OSError as e:
                    if _deadstate is not None:
                        self.returncode = _deadstate
                    elif e.errno == _del_safe.ECHILD:
                        # This happens if SIGCLD is set to be ignored or
                        # waiting for child processes has otherwise been
                        # disabled for our process.  This child is dead, we
                        # can't get the status.
                        # http://bugs.python.org/issue15756
                        # subprocess.run()函数会执行retcode = process.poll()来返回进程执行结果，但这里如果创建的进程已经死亡了，这里的状态就默认标记为0。
                        self.returncode = 0
                finally:
                    self._waitpid_lock.release()
            return self.returncode

```

## multiprocess模块
`multiprocess`模块也允许我们创建进程，但这个创建进程类型比`subprocess`模块更多样，可以支持`fork`、`spawn`、`forkserver`这三种模式。

## 其他
### spawn和fork
`spawn`进程是创建一个完全资源独立的进程，`fork`进程是创建一个子进程。
