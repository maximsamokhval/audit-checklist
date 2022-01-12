# Настройка 


- [x] Обратить внимание на флаг -d это место где хранятся сеансовые данные и файлы журнала регистрации
- [x] Время последнего перезапуска сеанса 1С. Проверить установленный интервал перезапуска сеансов. 
- [x] Выполнить анализ формата журнала регистрации. Правильно: по дням и "старый" формат
- [x] Всем регламентным заданиям установлены именованные пользователи
- [x] Исключить пользователей DefUsers
- [x] Время ожидания блокировки данных ( в секундах ) - должно быть равно  20 


!!! note

    Важно помнить, что у 1С тоже есть ограничения по количеству используемых ядер. Например, версия ПРОФ может использовать только 12 ядер и, если поставить ее на 24-ядерный сервер, то половина процессорных мощностей будет простаивать.
    Если на сервере больше 12 ядер и Вы хотите, чтобы 1С использовала их все, необходимо приобрести версию КОРП.


!!! info "Источники"
     - [x]  [Взломать за 60 секунд. Настройка безопасности](https://is1c.ru/career/blog/vzlomat-za-60-sekund/)