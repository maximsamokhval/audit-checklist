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



[Инфостарт. Источник](https://infostart.ru/1c/articles/1437774/)