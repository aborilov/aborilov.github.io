---
title: "Не показывать preview для больших архивов в ranger"
date: 2012-11-08T16:00:00+03:00
---

После того как я начал использовать `ranger`, мне понравилось что он быстро показывает превью для файлов, картинок, архивов. Но вот с архивами была проблема, когда просто перемещаешься по директории и натыкаешься на большой архив, `ranger` начинает его разархивировать, что бы получить список файлов и пока он не закончит, нельзя перемещаться дальше. Поэтому первое что пришло в голову, заставить `ranger` не показывать превью для архивов больше определенного размера. Как сделать это штатными средствами я не нашел. Да скорее всего это и не возможно. Потому что отображение preview занимается не сам `ranger`, а скрипт `scope.sh`, который мы можем свободно править. Находиться он в директории  
```
~/.config/ranger/scope.sh
```
на необходим вот этот кусок кода:
```
case "$extension" in
    # Archive extensions:
    7z|a|ace|alz|arc|arj|bz|bz2|cab|cpio|deb|gz|jar|lha|lz|lzh|lzma|lzo|\
    rar|rpm|rz|t7z|tar|tbz|tbz2|tgz|tlz|txz|tZ|tzo|war|xpi|xz|Z|zip)
        als "$path" | head -n $maxln
        success &amp;&amp; exit 0 || acat "$path" | head -n $maxln &amp;&amp; exit 3
```
в строке
```
als "$path" | head -n $maxln
```
мы и получаем список файлов из архива
Нам неоходимо это делать только вслучее если размер архива меньше опреденного.
Для этого вводи переменную
```
maxSize=20000000
```
что равно `20Мб`
и добавляем получение условие с размером архива, в итоге получаем:
```
maxSize=20000000
case "$extension" in
    # Archive extensions:
    7z|a|ace|alz|arc|arj|bz|bz2|cab|cpio|deb|gz|jar|lha|lz|lzh|lzma|lzo|\
    rar|rpm|rz|t7z|tar|tbz|tbz2|tgz|tlz|txz|tZ|tzo|war|xpi|xz|Z|zip)
        [ `stat -c%s $path` -lt $maxSize ] &amp;&amp; als "$path" | head -n $maxln
        success &amp;&amp; exit 0 || acat "$path" | head -n $maxln &amp;&amp; exit 3
```
сохраняем и все готов. Ни чего не надо перезапускать, все сразу работает.
