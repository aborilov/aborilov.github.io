---
title: "Виджет раскладки клавиатуры в QTile"
date: 2012-09-26T16:00:00+03:00
---

В Qtile мне очень не хватало аплета отображающего текущую раскладку клавиатуры и поэтому я решил озадачиться этим вопросам. В [прошлой статье](/post/keyboard_layout_python/) я описал как получить текущую раскладку клавиатуры в `Python`, теперь осталось прикрутить это к `qtile`.
Что оказалось совсем не сложно сделать. За основу я взял виджет часов, который является TextBox'ом. Весь Qtile у меня находить в директории
```
/usr/local/lib/python2.7/dist-packages/libqtile/
```
виджеты во вложеной директории widget  
Класс виджета часов выглядит следующим образом:
{{< highlight python >}}
import datetime
from .. import hook, bar, manager
import base

class Clock(base._TextBox):
    """
        A simple but flexible text-based clock.
    """
    defaults = manager.Defaults(
        ("font", "Arial", "Clock font"),
        ("fontsize", None, "Clock pixel size. Calculated if None."),
        ("padding", None, "Clock padding. Calculated if None."),
        ("background", "000000", "Background colour"),
        ("foreground", "ffffff", "Foreground colour")
    )
    def __init__(self, fmt="%H:%M", width=bar.CALCULATED, **config):
        """
            - fmt: A Python datetime format string.

            - width: A fixed width, or bar.CALCULATED to calculate the width
            automatically (which is recommended).
        """
        self.fmt = fmt
        base._TextBox.__init__(self, " ", width, **config)

    def _configure(self, qtile, bar):
        base._TextBox._configure(self, qtile, bar)
        self.timeout_add(1, self.update)

    def update(self):
        now = datetime.datetime.now().strftime(self.fmt)
        if self.text != now:
            self.text = now
            self.bar.draw()
        return True
{{< /highlight >}}
Как видно в методе update производиться присвоение значения текстового поля, в _configure прописывается обновлять виджет каждую секунду, что для нас является очень полезным. Скопируем данный класс под именем `kblayout.py`. Это и будет наш новый виджет. Так же в директорию с виджетами необходимо скопировать класс который занимется получением текущей раскладки клавиатуры из [прошлой статьи](/post/keyboard_layout_python/) под именем `kb.py`.  
После изменения `kblayout.py` получаем:
{{< highlight python >}}
from .. import hook, bar, manager
import base
import subprocess
from kb import kb


class kblayout(base._TextBox):
    """
        A simple but flexible text-based kblayout.
    """
    defaults = manager.Defaults(
        ("font", "Arial", "kblayout font"),
        ("fontsize", None, "kblayout pixel size. Calculated if None."),
        ("padding", None, "kblayout padding. Calculated if None."),
        ("background", "000000", "Background colour"),
        ("foreground", "ffffff", "Foreground colour")
    )

    def __init__(self, width=bar.CALCULATED, **config):
        base._TextBox.__init__(self, "ru", width, **config)

    def _configure(self, qtile, bar):
        base._TextBox._configure(self, qtile, bar)
        self.timeout_add(1, self.update)

    def update(self):
        k = kb()
        l = k.curkb()
        if self.text != l:
            self.text = l
            self.bar.draw()
        return True
{{< /highlight >}}
Тут у меня вылезлал какая то проблема с `gobject`, который используется в `qtile`, которую я так и не смог решить, но смог ее обойти. Для этого в `kb.py` добавляем строку
```
del sys.modules['gobject']
```
получается
{{< highlight python >}}
import sys
import os
import subprocess
os.environ['GI_TYPELIB_PATH'] =\
    'libxklavier:' + os.environ.get('GI_TYPELIB_PATH', '')

del sys.modules['gobject']
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

k = kb()
print k.curkb()
{{< /highlight >}}
И в классе виджета `mpriswidget.py` за меняем строку
{{< highlight python >}}
import gobject
{{< /highlight >}}
на
{{< highlight python >}}
from gi.repository import Xkl, Gdk, GdkX11
{{< /highlight >}}
Теперь проблем не должно возникнуть. Осталось прописать наш виджет в файл `__init__.py` директории `widget`
{{< highlight python >}}
from kblayout import kblayout
{{< /highlight >}}
и прописать наш виджет в своем конфиге qtile - `~/.config/qtile/config.py`  
У меня это выглядит вот так:
{{< highlight python >}}
screens = [
    Screen(
        top=bar.Bar([widget.GroupBox(
            borderwidth=2, font='Consolas', fontsize=13,
            padding=1, margin_x=1, margin_y=1),
            widget.Sep(),
            widget.WindowName(
                font='Consolas', fontsize=13, margin_x=6),
            widget.Sep(),
            widget.Battery(
                font='Consolas', fontsize=13, margin_x=6,
                energy_now_file="charge_now",
                energy_full_file="charge_full",
                power_now_file="current_now"),
            widget.Sep(),
            widget.Systray(),
            widget.kblayout(font='Consolas', fontsize=13, padding=6),
            widget.Sep(),
            widget.Clock(
                '%H:%M:%S %d.%m.%Y',
                font='Consolas', fontsize=13, padding=6), ], 24,),
    ),
]
{{< /highlight >}}
