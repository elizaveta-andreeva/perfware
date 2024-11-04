# perfware

[Задание](https://github.com/elizaveta-andreeva/perfware/blob/main/perfware_task.pdf)

## Тест файловой системы
Допустим, наша система предназначена для хранения данных и быстрого доступа к ним. Нам нужно измерить производительность хранилища данных. В данном случае тест IOPS будет информативнее и полезнее.

Будем использовать следующий тест:
```bash
[test1]
blocksize=4k
rw=randwrite
direct=1
buffered=0
ioengine=libaio
iodepth=16
runtime=30
directory=/home/liza
size=5G

[test2]
blocksize=4k
rw=randread
direct=1
buffered=0
ioengine=libaio
iodepth=16
runtime=30
directory=/home/liza
size=5G
```

## Измерения производительности
Для получения статистики нагрузки на IO диска выполним команду:
```bash
sudo perf record -e block:block_rq_issue -ag fio fio.job
```

И посмотрим на результат
```bash
sudo perf report
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 2M of event 'block:block_rq_issue'
# Event count (approx.): 2621595
#
# Children      Self  Command  Shared Object      Symbol
# ........  ........  .......  .................  ....................................
#
    99.99%    99.99%  fio      [kernel.kallsyms]  [k] blk_mq_start_request
    99.99%     0.00%  fio      libc.so.6          [.] syscall
            |
            ---syscall
               entry_SYSCALL_64_after_hwframe
               do_syscall_64
               __x64_sys_io_submit
               io_submit_one
               |
               |--50.00%--aio_read
               |          ext4_file_read_iter
               |          iomap_dio_rw
               |          __iomap_dio_rw
               |          blk_finish_plug
               |          blk_flush_plug_list
               |          blk_mq_flush_plug_list
               |          blk_mq_sched_insert_requests
               |          blk_mq_try_issue_list_directly
               |          blk_mq_request_issue_directly
               |          __blk_mq_try_issue_directly
               |          scsi_queue_rq
               |          blk_mq_start_request
               |          blk_mq_start_request
               |
                --50.00%--aio_write
                          ext4_file_write_iter
                          iomap_dio_rw
                          __iomap_dio_rw
                          blk_finish_plug
                          blk_flush_plug_list
                          blk_mq_flush_plug_list
                          blk_mq_sched_insert_requests
                          blk_mq_try_issue_list_directly
                          blk_mq_request_issue_directly
                          __blk_mq_try_issue_directly
                          scsi_queue_rq
                          blk_mq_start_request
                          blk_mq_start_request
...
```


Для записи измерений производительности выполним команду:
```bash
sudo perf record -ag fio fio.job
```

## Визуализация результатов
С помощью `gecko` создадим файл `gecko_profile.json` для последующей визуализации с помощью Firefox Profiler:
```bash
sudo perf script report gecko
```
Полученный график:
![](https://github.com/elizaveta-andreeva/perfware/blob/main/big_chart.png)

## Модификация gecko
Удалось сделать так, чтобы возле каждого потока отображалось к какому `cpu` он относился. Я предполагала, что будет возможность группировать потоки точно также по процессорам, как изначально по процессам, но сам Firefox Profiler во-первых всегда ищет `pid`, во-вторых непонятно какие данные ему передавать, чтобы он интерпретировал их как процессор (это могло быть условное поле "cpu", но он не интерпретирует его). 

![](https://github.com/elizaveta-andreeva/perfware/blob/main/modified.png)

Модифицированный код `gecko.py` находится [тут](https://github.com/elizaveta-andreeva/perfware/blob/main/gecko.py).
