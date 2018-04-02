---
title: iterm-lrzsz
date: 2018-03-21 09:50:21
tags: [ iterm , lrzsz ]
categories: tools
---

用过secureCRT，都知道lrzsz在服务器上上传下载文件有多方便。
这里有个工具[iterm2-zmodem](https://github.com/mmastrac/iterm2-zmodem.git)，简单配置下，也很实用


### Save the iterm2-send-zmodem.sh and iterm2-recv-zmodem.sh scripts in /usr/local/bin/

chmod +x /usr/local/bin/iterm2-send-zmodem.sh 
chmod +x /usr/local/bin/iterm2-recv-zmodem.sh 

### Preferences=>Profiles=>Advanced=>Triggers=>edit Set up Triggers in iTerm 2 like so:

```
Regular expression: rz waiting to receive.\*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh
Instant: checked

Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
Instant: checked
```
