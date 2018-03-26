---
title: "Получение текущей раскладки клавиатуры в python"
date: 2012-09-26T16:00:00+03:00
---

Я так и не смог найти нормального способа получить текущую раскладку клавиатуры из командрой строки. В интернете много разный примеров, но ни один у меня не выдал нужного результата(система Ubuntu 12.04). Единственное что дает хоть какой-то результат:  
```
xset -q
```
на выходе получаем
```
Keyboard Control:  
  auto repeat: on  key click percent: 0  LED mask: 00000002  
  XKB indicators:  
   00: Caps Lock:  off  01: Num Lock:  on   02: Scroll Lock: off  
   03: Compose:   off  04: Kana:    off  05: Sleep:    off  
   06: Suspend:   off  07: Mute:    off  08: Misc:    off  
   09: Mail:    off  10: Charging:  off  11: Shift Lock: off  
   12: Group 2:   off  13: Mouse Keys: off  
  auto repeat delay: 500  repeat rate: 33  
  auto repeating keys: 00ffffffdffffbbf  
             fadfffefffedffff  
             9fffffffffffffff  
             fff7ffffffffffff  
  bell percent: 50  bell pitch: 400  bell duration: 100  
 Pointer Control:  
  acceleration: 2/1  threshold: 4  
 Screen Saver:  
  prefer blanking: yes  allow exposures: yes  
  timeout: 600  cycle: 600  
 Colors:  
  default colormap: 0x20  BlackPixel: 0  WhitePixel: 16777215  
 Font Path:  
  /usr/share/fonts/X11/misc,/usr/share/fonts/X11/100dpi/:unscaled,/usr/share/fonts/X11/75dpi/:unscaled,/usr/share/fonts/X11/Type1,/usr/share/fonts/X11/100dpi,/usr/share/fonts/X11/75dpi,/var/lib/defoma/x-ttcidfont-conf.d/dirs/TrueType,built-ins  
 DPMS (Energy Star):  
  Standby: 0  Suspend: 0  Off: 0  
  DPMS is Enabled  
  Monitor is On  
```
Для нас важна вторая строка, а конкретно
```
LED mask: 00000002  
```
если мы переключим раскладку с default на другую то получим
```
LED mask: 00001002
```
вот у все что у нас есть. Если у вас больше двух раскладок, то вы не узнаете какая у вас сейчас. Данным способом мы можем узнать только дефолтная раскладка сейчас или нет. Что на самом деле достаточно для большенства.  
В `bash` можно использовать в таком виде
```
xset -q | grep LED | awk '{print $10}' | cut -c 5  
```
возвращает 0 при дефолтной раскладке и 1 при альтернативной.  
Но мне раскладка клавиатуры нужна в `Python`.  
Для получения группы раскладок в `Python` мы можем использовать следующий код:
{{< highlight python >}}
import sys

import os

os.environ['GI_TYPELIB_PATH'] = 'libxklavier:' + os.environ.get('GI_TYPELIB_PATH', '')

from gi.repository import Xkl, Gdk, GdkX11

display = GdkX11.x11_get_default_xdisplay()

engine = Xkl.Engine.get_instance(display)

print('group names:', engine.get_groups_names())

{{< /highlight >}}
Теперь имея список раскладок, мы можем поставить в соответствие значение получаемое из
```
xset -q  
```
с раскладкой из группы `engine.get_groups_names()`  

в итоге получился такой не большой класс:
{{< highlight python >}}
import sys
import sys
import os
import subprocess
os.environ['GI_TYPELIB_PATH'] =\
    'libxklavier:' + os.environ.get('GI_TYPELIB_PATH', '')

from gi.repository import Xkl, Gdk, GdkX11


class kb():

    def __init__(self):
        display = GdkX11.x11_get_default_xdisplay()
        self.engine = Xkl.Engine.get_instance(display)

    def curkb(self):
        t = subprocess.check_output(["xset", "-q"])
        num = int(t.splitlines()[1].split()[9][4])
        groups = self.engine.get_groups_names()
        if (len(groups) &gt; num):
            return str(groups[int(num)])[:2]
        else:
            return "Err"
{{< /highlight >}}
