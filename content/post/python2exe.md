---
title: "Компиляция python кода в exe под Linux"
date: 2012-12-28T10:44:16+03:00
draft: true
---
Если на linux машинах python установлен повсеместно, то windows все наоборот.
Решил скомпилировать свое приложения в exe, причем желательно что бы это был одинокий exe файл и не тянул с собой дополнительные dll библиотеки.  
Для компиляции python кода в бинарные файлы есть очень удобная программа [pyinstaller](http://www.pyinstaller.org).  
Поддержку cross компиляции они уже начали добавлять, но пока она работает не полноценно, поэтому я решил установить python под wine.  
Устанавливаем [wine](http://www.winehq.org).
Качаем [python](http://www.python.org/download/releases/).  
У меня, кстати, python 2.7.2 отказался устанавливать под wine, а вот 2.7.1 встал без проблем.  
Для работы `pyinstaller` под windows нужен пакет `pywin32`, качаем и устанавливаем([http://sourceforge.net/projects/pywin32/](http://sourceforge.net/projects/pywin32/").  
Качаем pyinstaller([http://www.pyinstaller.org/](http://www.pyinstaller.org/).  
После того как у нас установлен python и pywin32 под wine, мы готовы.  
Распаковываем pyinstaller куда-нибудь поближе к установленном python, у меня это было
/home/pavel/.wine/drive_c/Python27/progs/pyinstaller-1.5.1  
Туда же, в progs, копируем наше python приложение.  
Для начала надо что pyintaller создал файл конфигурации для нашей системы  
```
wine python.exe progs/pyinstaller-1.5.1/Configure.py
```
Далее создаем spec файл для нашего python приложения
```
wine python.exe progs/pyinstaller-1.5.1/Makespec.py -F progs/foobar.py</blockquote>
```
ключик `-F` как раз нужен для того чтобы на выходе у нас получился `stand-alone executables`  
`Makespec.py` сгенерирует файл `foobar.spec`, который нам необходим для компиляции.  
И последнее что осталось сделать, это сбилдить наш бинарник.  
```
wine python.exe progs/pyinstaller-1.5.1/Build.py foobar.spec
```
И в директории `dist` получаем наш `foobar.exe`.

