---
layout: post
title: "Vault: Данные под замком"
subtitle: "Инструмент ориентированный на безопасность и контроль данных"
date: 2019-12-23 13:37:00 +0300
background: '/img/posts/vault-compressor.jpg'
---
# Vault: Данные под замком

Множество приложений, баз данных, и иные сервисы имеют данные для доступа (Логин, пароль, сертификат и т.п.), 
количество сервисов растет и управлять всем становится сложнее, но как быть если необходимо выдать доступ 
к определенному сервису и на определенный период при это сохранить историю обращений к ключу? Одним из решений является
-[Vault](https://www.hashicorp.com/products/vault/).   

[Vault](https://www.hashicorp.com/products/vault/) позволяет решать такие проблемы как:
- Администрирование паролей, предоставить доступ лишь к тому что требуется.
- Возможность отозвать, продлить, перевыпустить ключ.
- Логирование, каждое обращение, изменение, обновление ключа будет записано системой. 
- Масштабируемость и доступность 
- Низкий уровень "входа" для понимания разработчиками и администраторами
- Безопасность, все ключи и пароли хранятся в зашифрованном виде
- Множество готовых решений для популярных систем
- ...Многое другое

### Установка

Vault распространяется под все популярные системы: 
- MAC OS X
- WINDOWS
- LINUX
- FREEBSD
- NETBSD
- OPENBSD
- SOLARIS

Загрузить его можно на [официальном сайте](https://www.vaultproject.io/downloads.html)

##### Установка на Windows 

После загрузки архива размером 50 MB~, в нем вы обнаружите один исполняемый файл. Наиболее простым решением является 
распаковать его в  `C:\Windows\system32`. Либо создать отдельную папку, распаковать в нее данный файл и прописать путь к 
папке через `"Переменные среды"` добавив путь в переменную `Path`.

##### Первые шаги

После того как Vault установлен время приступить к созданию своего первого ключа для этого стоит запустить сервера 
разработки Vault:

```powershell
> vault.exe server -dev
```

Сервер разработки будет хранить все ваши секреты в оперативной памяти, но клиент не 
сможет подключиться к нему, так как мы не обеспечили безопасное соединение (HTTPS), эту проблемы можно решить явно
указав клиенту протокол небезопасное соединение HTTP - `$env:VAULT_ADDR="http://127.0.0.1:8200"`. Первым шагом запишем
простой ключ в хранилище: 

```powershell
>  vault.exe kv put secret/KeyName hello=word 
Key              Value
---              -----
created_time     2019-12-23T21:10:22.6628931Z
deletion_time    n/a
destroyed        false
version          1
```

Если более подробно говорить о том что произошло: 
1) `kv` - Указывает на тип хранилища в нашей случае Key-Value(Ключ-Значение)
2) `put` - Команда указывающая на добавление
3) `secret/KeyName` - Имя "Коллекции"/"ИмяКлюча" куда сохранить. Коллекцию в данном случае, можно понимать как "базу 
данных" или "раздел" 
4) `hello=world` - hello - В данном случае является ключом, world - значение

Если вы попробуете повторно сделать запись ничего не изменив вы заметите что Vault повысит номер версии

```powershell
> vault.exe kv put secret/KeyName hello=world excited=yes
Key              Value
---              -----
created_time     2019-12-23T21:17:16.1989736Z
deletion_time    n/a
destroyed        false
version          2
```

Ваш первый ключ уже в хранилище! Но как получить его значение? Для этого нужно воспользоваться командой: 

```powershell
> vault.exe kv get secret/KeyName
====== Metadata ======
Key              Value
---              -----
created_time     2019-12-23T21:17:17.1989736Z
deletion_time    n/a
destroyed        false
version          2

==== Data ====
Key      Value
---      -----
hello    world
```

Я думаю вывод команды предельно понятен, но как получить лишь нужные данные, а именно значение нужного ключа? Для этого
используем явное указание интересующего поля: 

```powershell
> vault.exe kv get -field=hello secret/KeyName
world
```

Так же часто бывает полезным получить иной формат ответа, например json: 

```powershell
> vault.exe kv get -format=json  secret/KeyName
{
  "request_id": "423c17c0-23be-5a12-cd75-c9334926b685",
  "lease_id": "",
  "lease_duration": 0,
  "renewable": false,
  "data": {
    "data": {
      "hello": "world"
    },
    "metadata": {
      "created_time": "2019-12-23T21:21:00.082784Z",
      "deletion_time": "",
      "destroyed": false,
      "version": 2
    }
  },
  "warnings": null
}
```

Рассмотрим процесс удаления ключа из коллекции: 

```powershell
> vault.exe kv delete secret/KeyName
Success! Data deleted (if it existed) at: secret/KeyName
```


Стандартные операции мы рассмотрели, далее обсудим более тонкую настройку. Добавим свою "коллекцию"(хранилище):
 
```powershell
> vault.exe secrets enable -path=new_storage -description="My first storage" kv
Success! Enabled the kv secrets engine at: new_storage/
```

1) `secrets` - Обращаемся к команде 
2) `enable` - В данном случае данная команда создает новую, но может и активировать "выключенную" "коллекцию"
3) `-path` - Указываем имя нашей новой коллекции после знака `=`. Пробелы не допускаются.
4) `-description` - Указываем описание коллекции после знака `=`, но в двойных кавычках. Данная информация не 
используется системой, когда "коллекций" много это поможет вам сориентироваться.
5) `kv` - Указываем тип хранилища (Key-Value)

Теперь убедимся в том что новая "Коллекция" добавлена, активна и имеет установленный нами типа `kv`, для этого выведем 
список всех "коллекций":

```powershell
> vault.exe secrets list
Path            Type         Accessor              Description
----            ----         --------              -----------
cubbyhole/      cubbyhole    cubbyhole_09cd5a15    per-token private secret storage
identity/       identity     identity_d3c65804     identity store
new_storage/    kv           kv_52b4de30           My first storage
secret/         kv           kv_ea3eb018           key/value secret storage
sys/            system       system_e793490c       system endpoints used for control, policy and debugging
```

Удаление "коллекции" осуществляется командой `disable`:

```powershell
> vault.exe secrets disable new_storage
Success! Disabled the secrets engine (if it existed) at: new_storage/
``` 

Обратите внимание на уточнение `if it existed`, тойсть команда всегда будет успешной, даже если такого хранилища нет,
поэтому не забывайте проверить результат. На этом первая вводная часть закончена.
