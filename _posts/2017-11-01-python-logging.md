---
layout: post
title: python의 logging, logger, handler
---


이상한 모임 슬랙 #dev-python채널에서 @linusdamyo님께서

```python
import logging
logger = logging.getLogger('mylogger')
logger.setLevel(logging.DEBUG)


logger.debug("debug")
logger.info("info")
logger.warning("warning")
logger.error("error")
logger.critical("critical")

# 출력
# warning
# error
# critical
```
이 코드에서 `logger.setLevel(logging.DEBUG)`로 로그 레벨을 DEBUG로 낮췄는데 왜 debug와 info는 출력되지 않는가? 를 질문하셨는데 저도 궁금증이 생겨서 좀 찾아보았습니다.

python 로깅모듈 구조 상 로그를 처리하는건 결국 핸들러이기 때문에 핸들러가 없으면 아무 처리도 안되는게 정상입니다.

그렇다면 핸들러가 없는데 warning, error, critical은 왜 찍히는가? 라는 의문이 생기는데요,

기본 logger 클래스의 핸들러 호출 함수 구현을 보면 로그 레코드를 처리할 수 있는 핸들러가 없을 경우  lastResort 핸들러로 처리하는 시도를 합니다.


```python
    def callHandlers(self, record):
        """
        Pass a record to all relevant handlers.

        Loop through all handlers for this logger and its parents in the
        logger hierarchy. If no handler was found, output a one-off error
        message to sys.stderr. Stop searching up the hierarchy whenever a
        logger with the "propagate" attribute set to zero is found - that
        will be the last logger whose handlers are called.
        """
        c = self
        found = 0
        while c:
            for hdlr in c.handlers:
                found = found + 1
                if record.levelno >= hdlr.level:
                    hdlr.handle(record)
            if not c.propagate:
                c = None    #break out
            else:
                c = c.parent
        if (found == 0):
            if lastResort:
                if record.levelno >= lastResort.level:
                    lastResort.handle(record)
            elif raiseExceptions and not self.manager.emittedNoHandlerWarning:
                sys.stderr.write("No handlers could be found for logger"
                                 " \"%s\"\n" % self.name)
                self.manager.emittedNoHandlerWarning = True
```

기본 lastResort는 StderrHandler(WARNING)으로 만들어져 있고 이 핸들러는 WARNING레벨 이상의 로그 레코드를 Stderr로 출력하는 핸들러입니다.

```python
_defaultLastResort = _StderrHandler(WARNING)
lastResort = _defaultLastResort
```

따라서 아무런 핸들러가 없는 상태의 로거에 warning이나 error를 호출하면 lastResort 핸들러가 처리하게 되고 stderr에 메세지가 출력되게 됩니다.

lastResort의 처리 level은 로거의 setLevel 함수에 영향을 받지 않으므로 setLevel로 로거의 로깅 레벨을 낮추었다고 하더라고 lastResort는 warning 미만의 메시지는 처리하지 않습니다.


질문하신 코드와 비슷하지만 미묘하게 다른 다음 코드는 따로 핸들러를 안만들어줘도 setLevel이 동작하는 것처럼 보이는데요,

```python
import logging
logging.getLogger().setLevel(logging.DEBUG)


logging.debug("debug")
logging.info("info")
logging.warning("warning")
logging.error("error")
logging.critical("critical")

# 출력
# DEBUG:root:debug
# INFO:root:info
# WARNING:root:warning
# ERROR:root:error
# CRITICAL:root:critical
```

질문하신 코드는 `logger.debug()` 형태로 로거의 함수를 직접 호출했고, 위 코드는 `logging.debug()`형태로 로깅 모듈의 debug함수를 호출합니다.

logging모듈의 debug함수 구현을 보면 핸들러가 없는 경우 `basicConfig`를 호출해서 핸들러를 만들도록 되어있습니다.

```python
def debug(msg, *args, **kwargs):
    """
    Log a message with severity 'DEBUG' on the root logger. If the logger has
    no handlers, call basicConfig() to add a console handler with a pre-defined
    format.
    """
    if len(root.handlers) == 0:
        basicConfig()
    root.debug(msg, *args, **kwargs)
```
따라서 핸들러가 하나 생성이 되게 되고 setLevel로 지정한 level이 적용되어 debug와 info 로그 레코드도 처리되는 것을 볼 수 있습니다.

