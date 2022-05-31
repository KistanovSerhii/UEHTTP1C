# UEHTTP1C
Universal extension HTTP for 1C - Универсальное расширение HTTP для 1С

1. Создавая пользователя необходимо через конфигуратор и открыв в дереве конф. это расширение.
ИмяПользователя: ServiceUser Pass: цветок ИмяПользователяПолное: ServiceUser - Служебный (для HTTP и WEB сервисов)
При внедрении роль только одна "ITExe_ServiceDataExchange" (больше никаких ролей не надо)
2. Достаточно создать пользователя из под конфигуратора и дать ему исключительно
разработанную нами сервисную роль (дает работать с HTTP и читать/изменять нужные обк).

Это не даст запустить 1С интерактивно, но позволит пользоватся HTTP сервисами!

3. Прописать прокси проброс через nginx с указанием логина и пароля (поможет postman).
4. Перезагрузить IIS после публикации. И не забыть проверить наличие файла "web.config"
в папке публикации!
5. Перезагрузить Nginx после проброса.

Код проброса добавлял не в самом низу, а в середине, после:
"http {
    include       mime.types;"

* Все запросы идущие на um сервера 100 - будут перенаправлены на сервер 150 в папку публикации 1С
* Выполнив url: http://192.168.171.100:4000/um/hs/um/employee/ мы получим перенаправление на 150
* та часть которая идет после um будет подставленна к знч proxy то есть:
* http://192.168.171.100:4000/um/hs/um/employee/ => http://192.168.171.150/um/ + hs/um/employee/
* ( um стоящий перед hs - это каталог публикации 1C )
* где hs это зарезервированное слово, um (после hs) - это имя обк HTTP, employee - http метод
        location /um {
            proxy_pass http://192.168.171.150/um/;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real-IP $remote_addr;
            
            # "Basic U2VydmljZVVzZXI6UjBtYXNoa2E=" - это пароль полученный программой postman
            proxy_set_header Authorization "Basic U2VydmljZVVzZXI6UjBtYXNoa2E=";
            proxy_pass_header Authorization;
        }

        location /uat {
            proxy_pass http://192.168.171.150/uat/;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real-IP $remote_addr;
            
            # "Basic U2VydmljZVVzZXI6UjBtYXNoa2E=" - это пароль полученный программой postman
            proxy_set_header Authorization "Basic U2VydmljZVVzZXI6UjBtYXNoa2E=";
            proxy_pass_header Authorization;
        }
        
        
        
# Как работает переадресация nginx

Nginx.conf:
* все что прийдет на сервер nginx на адрес "NginxServer_IP/uat/"
location /uat/ {

# будет перенаправленно на этот адрес, а это
# папка в которую опубликована база 1С (УАТ)
            proxy_pass http://192.168.171.150/uat/


Пример:
http://192.168.171.100:4000/uat/hs/VMDE/Employee/УА00000071

* "hs/VMDE/Employee/УА00000071" - эта часть перенаправится на "http://192.168.171.150/uat/"

* hs 		- Зарезервированное слово "http service"
* VMDE		- Знч корневого url у прикладного объекта HTTP сервис
* Employee	- ВАЖНЫЙ момент, это слово которое прописанно в шаблоне реквизита "http"
, а точней там прописанна строка "/Employee/{Code}" (значит что ожидает параметр "Code")
ТАМ НЕЛЬЗЯ ОСТАВЛЯТЬ /* ТАК КАК ВСЕ ЗАПРОСЫ ДАЛЬШЕ ЧЕМ ЭТОТ ЗАПРОС НЕ УЙДУТ!!!


![HTTPUrlFormula](https://user-images.githubusercontent.com/28355711/171154776-0e6d7955-ce9d-4954-a049-ca1f8e92fffa.png)

![HTTPUrlFormula2](https://user-images.githubusercontent.com/28355711/171154798-0a2e4df3-0375-4079-8c42-18269eb86946.png)

![HTTPРазрешениеНаИспользованиеОбработчикаHTTP_АТакжеНеобходимоДатьРазрешениеЧтениеЗаписьНаОбъектыСКоторымиРаботаем](https://user-images.githubusercontent.com/28355711/171154871-1c7de007-fe44-4c4d-9710-d56cde2869c9.png)

![HTTPПубликацияСервисаКоторыйСозданВРасширении](https://user-images.githubusercontent.com/28355711/171154887-5447aac5-c75b-464e-8e9c-1bccc08d4164.PNG)
