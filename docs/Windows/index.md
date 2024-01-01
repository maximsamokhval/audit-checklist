# Настройка 

<details>  
  <summary>Проверка установленной схемы электропитания</summary>

```  
powercfg /query SCHEME_CURRENT SUB_PROCESSOR PROCTHROTTLEMAX
```
</details>

- [x] Установить дополнительное программное обеспечение
    - [x] choco install sysinternals
    - [x] wget https://github.com/oscript-library/ovm/releases/download/v1.0.0-
RC15/ovm.exe
- [x] [oscript.io](https://oscript.io/downloads)

- [x] Проверка служб автозапуска 
  

В операционных системах Windows начиная с Windows Vista в конфигурации по умолчанию у протокола IPv6 имеется приоритет над протоколом IPv4. В некоторых ситуациях наличие этого приоритета может проявляться даже несмотря на то, что в привычном всем графическом интерфейсе настроек сетевого подключения протокол IPv6 выключен

``` cmd 

reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters" /v DisabledComponents /t REG_DWORD /d 0x20 /f

```

``` cmd
netsh interface ipv6 show prefixpolicies
```



Столбец Precedence определяет приоритет префикса. Чем больше число, тем выше приоритет

Префикс ::ffff:0:0/96 относится ко всем IPv4 адресам. В списке политик этот префикс представлен с меткой Label 4 и имеет более низкий приоритет по отношению ко всем IPv6 адресам (::/0)

После внесения изменений в реестр и перезагрузки сервера ситуация с приоритетами изменится так, что префикс, относящийся ко всем IPv4 адресам, поднимется на самый верхний уровень.


Недостатком описанного способа изменения приоритезации является необходимость перезагрузки системы для вступления изменений в силу. Если мы не хотим действовать через реестр и перезагружать систему, то есть альтернативный вариант повышения приоритета IPv4 над IPv6, который начинает работать сразу в оперативном режиме. Для этого нужно воспользоваться утилитой netsh

Чтобы поднять приоритет адресов IPv4 над всеми прочими, задав ему приоритет, например, 55, достаточно будет выполнить одну команду типа:


``` cmd 

netsh interface ipv6 set prefix ::ffff:0:0/96 55 4

```

## Запуск RAS как службы


``` cmd 

@echo off
@chcp 65001

rem после запуска, выбрать пользователя/пароль и запустить службу


rem %1 – полный номер версии 1С:Предприятия
set v8ver=8.3.20.1549
set CtrlPort=1540
set AgentName=localhost
set RASPort=1545
set SrvcName="1C:Enterprise 8.3 Remote Server"
set BinPath="\"C:\Program Files\1cv8\%v8ver%\bin\ras.exe\" cluster --service --port=%RASPort% %AgentName%:%CtrlPort%"
set Desctiption="Сервер администрирования 1С:Предприятия 8.3"
rem sc stop %SrvcName%
rem sc delete %SrvcName%
sc create %SrvcName% binPath= %BinPath% start= auto displayname= %Desctiption%

```



## Настройка сборки данных в Performance Monitor Windows Server

Для того, чтобы увидеть список всех счетчиков производительности, имеющихся на текущем компьютере нужно в командной строке выполнить

``` cmd 

typeperf -q [object] выведет список всех счетчиков	
typeperf -qх [object] выведет список всех счетчиков по экземплярам оборудования, например отдельно для дисков А: и С: 	

```

Где необязательный параметр [object] это фильтр по виду счетчиков, например PhysicalDisk
Этот вывод можно переадресовать в файл и далее уже из него выбирать необходимое


Создание счетчика 
В файл assembled.txt добавлять названия счетчиков. По одному на строку. ( можно получить из typeperf -q )

``` 

\Process(1cv8*)\% Processor Time
\Process(1cv8*)\Private Bytes
\Process(1cv8*)\Virtual Bytes
\Process(ragent*)\% Processor Time
\Process(ragent*)\Private Bytes
\Process(ragent*)\Virtual Bytes
\Process(rphost*)\% Processor Time
\Process(rphost*)\Private Bytes
\Process(rphost*)\Virtual Bytes
\Process(rmngr*)\% Processor Time
\Process(rmngr*)\Private Bytes
\Process(rmngr*)\Virtual Bytes
\Processor(_Total)\% Processor Time
\Processor(_Total)\Interrupts/sec
\LogicalDisk(_Total)\Free Megabytes
\Memory\Pages Output/sec
\Process(_Total)\% Processor Time
\Network Interface(*)\Bytes Total/sec
\System\Processor Queue Length
\PhysicalDisk(*)\Avg. Disk Queue Length
\PhysicalDisk(*)\Avg. Disk Write Queue Length
\PhysicalDisk(*)\Avg. Disk Read Queue Length
\Processor(_Total)\Interrupts/sec
\System\Context Switches/sec
\System\File Read Bytes/sec
\System\Processor Queue Length
\System\Threads


```

### Скрипт создания счетчика сбора 

``` cmd

chcp 65001	
logman create counter live_is_counter -f bincirc 	
logman update counter live_is_counter -cf assembled.txt 	
logman update counter live_is_counter -si 5 -v mmddhhmm	

```

### Показатели и их значения 


| **Счетчик** | **Предельные значения** |
| --- | --- |
| \\Processor(_Total)\\% Processor Time | Не более 70% — 80% в течение длительного времени, под длительным обычно понимается +10 минут. 50% — нормальная загрузка сервера |
| \\Processor(_Total)\\% User Time | Учитывая тот факт, что % Processor Time = % User Time + % Privileged Time, то в идеале значения % User Time должны стремиться к % Processor Time, а доля % Privileged Time стремиться к 0 |
| \\Processor(_Total)\\% Privileged Time | Норма % Privileged Time составляет 5-10%, о проблемах говорят значения \>20%. Обычно это проблема с драйверами |
| \\Memory\\Available MBytes | Постоянное и равномерное уменьшение счетчика указывает на утечку памяти в одном из приложений. Желательное состояние — 25% от общей памяти |
| \\Memory\\Pages/sec | По мнению 1С Максимальное: не более 20, в сети встречаются допустимые варианты и в 1000. Рассматривается совместно с Memory Available. Желательное состояние около 0. |
| \\Memory\\% Committed Bytes In Use | Не должен превышать размер оперативной памяти у Филлипова, но это видимо опечатка. *Memory % Committed Bytes in Use* представляет собой соотношение величин Memory/Committed Bytes и Memory\\Commit Limit исчисляется в процентах и должен быть менее 90%, больше 95% появится вероятность возникновения ошибки OutOfMemory. |
| \\Paging File(\*)\\% Usage | Рассматривается совместно с предыдущими счетчиками, по мнению Microsoft при  остальных штатных значениях 100% возможный вариант, желательное значение от 50 до 75% |
| \\System\\Context Switches/sec | Высокое значение — более 5000 переключений контекста/с.<br />Очень высокое значение — более 15000 переключений контекста/с |
| \\System\\Processor Queue Length | Не более 2 \* количество ядер процессоров в течение длительного времени |
| \\System\\Processes | Служит для построения базовой линии загрузки сервера |
| \\System\\Threads | Служит для построения базовой линии загрузки сервера |
| \\PhysicalDisk(_Total)\\Current Disk Queue Length | В статье ИТС: Не более 2 \* количество дисков, работающих параллельно, в основном его не рекомендуют больше использовать из-за виртуализации.<br />Из-за изменений в технологиях, таких как виртуализация, технология дисков и контроллеров, SAN и многое другое, этот счетчик больше не является хорошим индикатором узких мест ввода-вывода. |
| \\PhysicalDisk(\*)\\Current Disk Queue Length | Аналогично |
| \\PhysicalDisk(_Total)\\Avg. Disk sec/Read | Этот показатель рекомендуется рассматривать как замену Current Disk Queue Lengt. Для дисков с файлами MDF и NDF и загрузкой OLTP средняя задержка записи в идеале должна быть ниже 20 мс. Для дисков с нагрузкой OLAP приемлемой считается задержка до 30 мс. Для дисков с файлами LDF задержка в идеале должна составлять 5 мс или меньше. В общем, все, что превышает 50 мс, является медленным и предполагает потенциально серьезное узкое место ввода-вывода. У 1С — не более 50-200 мс. |
| \\PhysicalDisk(_Total)\\Avg. Disk sec/Write | Аналогично<br /> |
| \\Network interface(_Total)\\Bytes Total/sec | Не более 65% от пропускной способности сетевого адаптера |
| \\Network interface(_Total)\\Current Bandwidth | Network utilization **=** 8 \* \\Network interface(_Total)\\Bytes Total/sec / (Network Interface: Current Bandwidth) \*100 |
|  |  |
| \\SQLServer:General Statistics\\User Connections | Служит для построения базовой линии загрузки сервера |
| \\SQLServer:General Statistics\\Processes blocked | В идеале близок к 0 |
| \\SQLServer:Buffer Manager\\Buffer cache hit ratio | Если на вашем сервере запущены приложения для онлайн-обработки транзакций (OLTP), а это как раз торговые базы 1С, значение 99% или выше является идеальным, но все, что выше 90%, обычно считается удовлетворительным. Значение 90% или ниже может указывать на увеличенный доступ к вводу-выводу и более низкую производительность. |
| \\SQLServer:Buffer Manager\\Page life expectancy | Некоторые говорят, что значение ниже 300 (или 5 минут) означает, что вам может потребоваться дополнительная память. |
| \\SQLServer:SQL Statistics\\Batch Requests/sec | Служит для построения базовой линии загрузки сервера и совместно с другими показателями |
| \\SQLServer:SQL Statistics\\SQL Compilations/sec | В идеале в 10 раз меньше Batch Requests/sec |
| \\SQLServer:SQL Statistics\\SQL Re-Compilations/sec | В идеале в 10 раз меньше SQL Compilations/sec |
| \\SQLServer:Access Methods\\Page Splits/sec | В идеале меньше чем 20% от Batch Requests/sec |
| \\SQLServer:Access Methods\\Forwarded Records/sec | Служит для построения базовой линии загрузки сервера. Помогает понять, насколько фрагментированы таблицы SQL Server без кластерного индекса, не должен устойчиво расти со временем. |
| \\SQLServer:Access Methods\\Full Scans/sec | Служит для построения базовой линии загрузки сервера, при самописных конфигурациях или тюнинге системы используется совместно с  Index searches/sec |
| \\SQLServer:Memory Manager\\Target Server Memory (KB) | Служит для построения базовой линии загрузки сервера |
| \\SQLServer:Memory Manager\\Total Server Memory (KB) | Служит для построения базовой линии загрузки сервера |
| \\SQLServer:Memory Manager\\Free Memory (KB) | Служит для построения базовой линии загрузки сервера |
| \\SQLServer:Databases(_Total)\\Transactions/sec | Служит для построения базовой линии загрузки сервера |
| \\SQLServer:Databases(\*)\\Transactions/sec | Служит для построения базовой линии загрузки сервера |
|  |  |
| \\Process(1cv8)\\% Processor Time<br />\\Process(1cv8)\\Private Bytes<br />\\Process(1cv8)\\Virtual Bytes<br />\\Process(ragent)\\% Processor Time<br />\\Process(ragent)\\Private Bytes<br />\\Process(ragent)\\Virtual Bytes<br />\\Process(rphost)\\% Processor Time<br />\\Process(rphost)\\Private Bytes<br />\\Process(rphost)\\Virtual Bytes<br />\\Process(rmngr)\\% Processor Time<br />\\Process(rmngr)\\Private Bytes<br />\\Process(rmngr)\\Virtual Bytes<br />\\Process(sqlservr)\\% Processor Time<br />\\Process(sqlservr)\\Private Bytes<br />\\Process(sqlservr)\\Virtual Bytes | Служит для построения базовой линии загрузки сервера |


[Инфостарт. Источник](https://infostart.ru/1c/articles/1437774/)