<!--
author: chiefyang
head: http://pingodata.qiniudn.com/jockchou-avatar.jpg
date: 2015-07-31
title: LINUX PHP C++扩展开发
tags: php
category: php
status: publish
summary: LINUX PHP C++扩展开发
-->

PHP的pcntl多进程

PHP使用PCNTL系列的函数也能做到多进程处理一个事务。比如我需要从数据库中获取80w条的数据，再做一系列后续的处理，这个时候，用单进程？你可以等到明年今天了。。。所以应该使用pcntl函数了。

假设我想要启动20个进程，将1-80w的数据分成20份来做，主进程等待所有子进程都结束了才退出：

```php
$max = 800000;
$workers = 20;
 
$pids = array();
for($i = 0; $i < $workers; $i++){
    $pids[$i] = pcntl_fork();
    switch ($pids[$i]) {
        case -1:
            echo "fork error : {$i} \r\n";
            exit;
        case 0:
            $param = array(
                'lastid' => $max / $workers * $i,
                'maxid' => $max / $workers * ($i+1),
            );
            $this->executeWorker($input, $output, $param);
            exit;
        default:
            break;
    }
}
 
foreach ($pids as $i => $pid) {
    if($pid) {
        pcntl_waitpid($pid, $status);
    }
}
```
这里当pcntl_fork出来以后，会返回一个pid值，这个pid在子进程中看是0，在父进程中看是子进程的pid（>0），如果pid为-1说明fork出错了。

使用一个$pids数组就可以让主进程等候所有进程完结之后再结束了