# Лайфхаки

## Восстановление базы данных после динамического обновления


<details>  
  <summary>Скрипт очистки данных после динамического обновления</summary>

baza - Имя вашей базы данных

``` sql
delete from baza.dbo.config where FileName = 'commit'
delete from baza.dbo.config where FileName = 'dynamicCommit'
delete from baza.dbo.config where FileName = 'dbStruFinal'
```
</details>
