# perfware

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
Выполним команду 
```bash
sudo perf record -e block:block_rq_complete -a fio fio.job
```
И посмотрим на результат
```bash
$ sudo perf script
         swapper       0 [000]  6062.527812: block:block_rq_complete: 8,16 WS () 52556360 + 8 [0]
         swapper       0 [000]  6062.527829: block:block_rq_complete: 8,16 WS () 53635272 + 8 [0]
         swapper       0 [000]  6062.527839: block:block_rq_complete: 8,16 R () 61578424 + 8 [0]
         swapper       0 [000]  6062.527893: block:block_rq_complete: 8,16 R () 57951224 + 8 [0]
         swapper       0 [000]  6062.527897: block:block_rq_complete: 8,16 WS () 49722312 + 8 [0]
         swapper       0 [000]  6062.527900: block:block_rq_complete: 8,16 R () 58263560 + 8 [0]
         swapper       0 [000]  6062.527904: block:block_rq_complete: 8,16 WS () 45426280 + 8 [0]
         swapper       0 [000]  6062.527907: block:block_rq_complete: 8,16 R () 58306288 + 8 [0]
         swapper       0 [000]  6062.527909: block:block_rq_complete: 8,16 R () 62572648 + 8 [0]
         swapper       0 [000]  6062.527913: block:block_rq_complete: 8,16 R () 61750224 + 8 [0]
         swapper       0 [000]  6062.527916: block:block_rq_complete: 8,16 WS () 49008208 + 8 [0]
         swapper       0 [000]  6062.527919: block:block_rq_complete: 8,16 R () 60029528 + 8 [0]
         swapper       0 [000]  6062.527922: block:block_rq_complete: 8,16 R () 55691968 + 8 [0]
         swapper       0 [000]  6062.527924: block:block_rq_complete: 8,16 WS () 48783632 + 8 [0]
         swapper       0 [000]  6062.527927: block:block_rq_complete: 8,16 WS () 53603232 + 8 [0]
         swapper       0 [000]  6062.527929: block:block_rq_complete: 8,16 R () 57184280 + 8 [0]
         swapper       0 [000]  6062.527932: block:block_rq_complete: 8,16 WS () 54367416 + 8 [0]
         swapper       0 [000]  6062.527973: block:block_rq_complete: 8,16 WS () 48500784 + 8 [0]
         swapper       0 [000]  6062.527976: block:block_rq_complete: 8,16 R () 56165576 + 8 [0]
         swapper       0 [000]  6062.527979: block:block_rq_complete: 8,16 WS () 45494544 + 8 [0]
         swapper       0 [000]  6062.527982: block:block_rq_complete: 8,16 R () 59324408 + 8 [0]
         swapper       0 [000]  6062.527985: block:block_rq_complete: 8,16 WS () 53761560 + 8 [0]
         swapper       0 [000]  6062.527987: block:block_rq_complete: 8,16 WS () 51621672 + 8 [0]
         swapper       0 [000]  6062.527990: block:block_rq_complete: 8,16 WS () 54530304 + 8 [0]
         swapper       0 [000]  6062.527992: block:block_rq_complete: 8,16 WS () 47382616 + 8 [0]
         swapper       0 [000]  6062.527994: block:block_rq_complete: 8,16 WS () 48909376 + 8 [0]
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
[]()

## Модификация gecko
Удалось сделать так, чтобы возле каждого потока отображалось к какому `cpu` он относился. Я предполагала, что будет возможность группировать потоки точно также по процессорам, как изначально по процессам, но сам Firefox Profiler во-первых всегда ищет `pid`, во-вторых непонятно какие данные ему передавать, чтобы он интерпретировал их как процессор (это могло быть условное поле "cpu", но он не интерпретирует его). 

[](https://github.com/elizaveta-andreeva/perfware/blob/main/modified.png)

Модифицированный код `gecko.py` находится [тут](https://github.com/elizaveta-andreeva/perfware/blob/main/gecko.py).
