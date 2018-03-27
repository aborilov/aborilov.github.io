---
title: "Переключение трека в mpd через dmenu и mpc"
date: 2014-03-06T16:00:00+03:00
---

Если вы пользуетесь `MPD` и не хотите постоянно держать запущенным какой-нибудь клиент, типа `Sonata`, для переключения треков, то можно повесить на hotkey вот такой простой скрипт  
```
mpc playlist | dmenu -l 10 | xargs -I '{}' sh -c "mpc playlist | grep -rne '{}' | awk -F: '{print \$1}'" | xargs -I '{}' sh -c "test -n '{}'&amp;&amp; mpc play '{}'"
```
получаем текущей playlist
```
mpc playlist
```
и выводим его с помощью dmenu(вертикально, 10 строчек)
```
dmenu -l 10
```
посылаем выбранный трек
```
xargs -I '{}' sh -c
```
фильтруем playlist по выбранному треку и выставляем номер строки данного трека
```
mpc playlist | grep -rne '{}'
```
оставляем только номер трека
```
awk -F: '{print \$1}'"
```
проверяем что трек был выбран. В случае если нажали Esc, трек не надо переключать. Запускаем выбранные трек
```
xargs -I '{}' sh -c "test -n '{}'&amp;&amp; mpc play '{}'"
```
У меня в `Qtile` hotkey выглядит вот так:
{{< highlight python >}}
Key(
        [mod], "m",
        lazy.spawn(
            """
            mpc playlist | dmenu -l 10  |\
            xargs -I '{}' sh -c "mpc playlist \
            | grep -rne '{}' | awk -F: '{print \$1}'" \
            | xargs -I '{}' sh -c "test -n '{}' &amp;&amp; mpc play '{}'"
            """)
    ),
{{< /highlight >}}
Переключение вперед/назад/стоп
{{< highlight python >}}
Key([mod], "Home", lazy.spawn("mpc toggle")),
Key([mod], "End", lazy.spawn("mpc stop")),
Key([mod], "Page_Down", lazy.spawn("mpc next")),
Key([mod], "Delete", lazy.spawn("mpc prev")),
{{< /highlight >}}
