# С чего начать

### Что такое Duyler?

Duyler - это каркас для создания многофункциональных, высокопроизводительных приложений на языке PHP. 
Область применения Duyler весьма обширна: консольные приложения, веб-приложения, воркеры, инструменты для тестирования и многое другое. Всё это можно эффективно реализовать с помощью "Duyler" прямо сейчас. Всё что нужно, это установить Duyler, запустить Duyler и начать думать как Duyler!  

### Установка

Для установки потребуется локально установленный Docker

```shell
curl -L -O https://github.com/duyler/duyler/archive/refs/heads/master.zip
```
```shell
unzip master.zip -d duyler
```
```shell
cd duyler
```
```shell
make build
```
```shell
make up
```
По-умолчанию, будет создано и запущено приложение для работы с веб. Оно будет доступно по адресу [http://localhost/](http://localhost/)

### Общая схема работы

<img src="https://duyler.com/assets/img/action-bus.svg" width="100%">