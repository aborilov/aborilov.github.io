---
title: "Оконный менеджер Qtile"
date: 2014-06-03T14:15:00+03:00
---

[QTile](http://qtile.org/) это tiling оконный менеджер написанный полностью на `Python`. Основные его плюсы состоят в том что написан на `Python` и вроде как сейчас активно развивается. Конфиг тоже пишется на `Python`, поэтому для человека знакомого с данным языком проблем с конфигурацией не должно быть.  
Установить `Qtile` можно из `git` репозитория, репозиториев дистрибутивов, с помощью питоновский утилит `pip` и `easy_install`, а так же и из исходных кодов.  
Если вы ставили `Qtile` из репозитория дистрибутива, то на экране login должен быть доступен сеанс `Qtile`, в него и загружаемся.  
Пока у вас нету своего конфига, будет работать [DefaltConfig](https://github.com/qtile/qtile/blob/develop/libqtile/resources/default_config.py). Можно сохранить его в `~/.config/qtile/config.py` и начать править.  
Переключение между рабочими столами происходит по кнопке
```
CMD+буква+(a, s, d, f, u, i, o, p)
```  
Запуск терминала
```
CMD+Enter
```
Что бы запустить любой приложение, можно нажать `CMD+r`, появится строка ввода, куда надо вводить имя программы для запуска.  

Завершение `Qtile` по нажатию `WinKey+Ctrl+q`.  

Перезагрузка `Qtile` по нажатию `WinKey+Ctrl+r`.(удобно пока экспериментируешь с конфигом)  

По сути, это все что вам необходимо для того что бы начать создавать свой конфиг. На сайте разработчика есть небольшая документация, в которой освещены основные моменты настройки. Я описывать тоже самое не будет, просто приведу свой [конфиг](https://raw.githubusercontent.com/aborilov/code/master/qtile/config/config.py).

Разберем не которые детали:

выставляем клавиши
{{< highlight python >}}
mod = "mod4"
alt = "mod1"
{{< /highlight >}}

Управление музыкой через mpc
{{< highlight python >}}
    Key([mod], "Home", lazy.spawn("mpc toggle")),
    Key([mod], "End", lazy.spawn("mpc stop")),
    Key([mod], "Page_Down", lazy.spawn("mpc next")),
    Key([mod], "Delete", lazy.spawn("mpc prev")),
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
закрыть текущее окно
{{< highlight python >}}
    Key(
        [mod], "w",
        lazy.window.kill()
    ),
{{< /highlight >}}
поднимаем спрятавшееся окно наружу(grave -&nbsp; это клавиша ~)<br />
{{< highlight python >}}
    Key(
        [alt], "grave",
        lazy.window.bring_to_front()
    ),
{{< /highlight >}}
переключение на следующее окно в группе(полезно когда какой-нибудь всплывающее окно ушло за задний фон)
{{< highlight python >}}
    Key(
        [alt], "Tab",
        lazy.group.next_window()),
{{< /highlight >}}
Переключение между группами
{{< highlight python >}}
    Key(
        [mod], "Left",
        lazy.group.prevgroup()
    ),
    Key(
        [mod], "Right",
        lazy.group.nextgroup()
    ),
{{< /highlight >}}
переключение между слоями
{{< highlight python >}}
    Key(
        [mod], "h",
        lazy.layout.previous()
    ),
    Key(
        [mod], "l",
        lazy.layout.next()
    ),
    Key(
        [mod], "j",
        lazy.layout.up()
    ),
    Key(
        [mod], "k",
        lazy.layout.down()
    ),
{{< /highlight >}}
переключить окно в плавающее(не обходимо для окон работающий не навесь экран, например диалоги)
{{< highlight python >}}
    Key(
        [mod], "f",
        lazy.window.toggle_floating()
    ),
{{< /highlight >}}
различные переключения и изменения размеров для разных layout. проще попробовать самому.
{{< highlight python >}}
    Key(
        [mod, "shift"], "space",
        lazy.layout.rotate(),
        lazy.layout.flip(),              # xmonad-tall
        ),
    Key(
        [mod, "shift"], "k",
        lazy.layout.shuffle_up(),         # Stack, xmonad-tall
        ),
    Key(
        [mod, "shift"], "j",
        lazy.layout.shuffle_down(),       # Stack, xmonad-tall
        ),
    Key(
        [mod, "control"], "l",
        lazy.layout.add(),                # Stack
        lazy.layout.increase_ratio(),     # Tile
        lazy.layout.maximize(),           # xmonad-tall
        ),
    Key(
        [mod, "control"], "h",
        lazy.layout.delete(),             # Stack
        lazy.layout.decrease_ratio(),     # Tile
        lazy.layout.normalize(),          # xmonad-tall
        ),
    Key(
        [mod, "control"], "k",
        lazy.layout.shrink(),             # xmonad-tall
        lazy.layout.decrease_nmaster(),   # Tile
        ),
    Key(
        [mod, "control"], "j",
        lazy.layout.grow(),               # xmonad-tall
        lazy.layout.increase_nmaster(),   # Tile
        ), 
{{< /highlight >}}
запуск различных программ
{{< highlight python >}}
    Key(
        [mod], "n",
        lazy.spawn("firefox")
        ),
    Key(
        [mod], "d",
        lazy.spawn("subl")
        ),

    Key(
        [mod], "u",
        lazy.spawn("uzbl-tabbed")
        ),
{{< /highlight >}}
{{< highlight python >}}
Key([mod], "Return", lazy.spawn("urxvt"))
{{< /highlight >}}
переключение на следующий layout
{{< highlight python >}}
Key([mod], "Tab",    lazy.nextlayout())
{{< /highlight >}}
запуск приложение через dmenu
{{< highlight python >}}
   Key(
        [mod], "space",
        lazy.spawn(
            "dmenu_run -fn 'Terminus:size=8' -nb '#000000' -nf '#fefefe'")
    ),
{{< /highlight >}}
настройка громкости
{{< highlight python >}}
    Key(
        [mod], "equal",
        lazy.spawn("amixer -c 0 -q set Master 2dB+")
    ),
    Key(
        [mod], "minus",
        lazy.spawn("amixer -c 0 -q set Master 2dB-")
    ),
{{< /highlight >}}
управление мышкой
{{< highlight python >}}
mouse = [
    Drag(
        [mod], "Button1",
        lazy.window.set_position_floating(),
        start=lazy.window.get_position()
    ),
    Drag(
        [mod], "Button3",
        lazy.window.set_size_floating(),
        start=lazy.window.get_size()
    ),
    Click(
        [mod], "Button2",
        lazy.window.bring_to_front()
    ),
]
{{< /highlight >}}
задаем рабочие столы. У меня это клавиши от 1 до 0. Для переключение нажимаем клавишу `mod` + номер. Для переноса окна на другой стол, `mod+shift+номер`  
{{< highlight python >}}
groups = [Group(str(i)) for i in (1, 2, 3, 4, 5, 6, 7, 8, 9, 0)]
for i in groups:
    keys.append(Key([mod], i.name, lazy.group[i.name].toscreen()))
    keys.append(
        Key(
            [mod, "shift"], i.name, lazy.window.togroup(i.name),
            lazy.group[i.name].toscreen()))
{{< /highlight >}}
настраиваем border для окон
{{< highlight python >}}
border = dict(
    border_normal='#808080',
    border_width=1,
)
{{< /highlight >}}
задаем какие будем использоваться layouts. [Здесь](http://docs.qtile.org/en/latest/manual/ref/layouts.html) можно почитать про встроенные `layouts`, но документация как всегда не успевает за разработчиками, так что лучше все посмотреть в [git](https://github.com/qtile/qtile/tree/develop/libqtile/layout)
{{< highlight python >}}
layouts = [
    layout.Max(),
    layout.MonadTall(**border),
    layout.Matrix(**border)
]
{{< /highlight >}}
теперь задаем наш экран. у меня один монитор, поэтому screen один. Можно настраивать поведение при нескольких мониторах. В нашем мониторе создаем `Bar` и задаем что на необходимо там видеть. Опять же, вот [тут](http://docs.qtile.org/en/latest/manual/ref/widgets.html) про встроенные виджеты или [тут](https://github.com/qtile/qtile/tree/develop/libqtile/widget) в исходных кодах.  
В вкратце, у меня используется `GroupBox` - отображает рабочие столы, `WindowName` - отображает заголовок текущего окна, `Notify` - типа статусной строки, `Systray` - системный трай, `KeyboardLayout` - раскладка клавиатуры, `Volume` - громкость, `BitcoinTicker` - все ясно, и часы. У каждого виджета есть свои настройки, более подробно в исходнике виджета.  
{{< highlight python >}}
screens = [
    Screen(
        top=bar.Bar(
            [
                widget.GroupBox(margin_y=1,
                                margin_x=1,
                                borderwidth=1,
                                padding=1,),
                widget.WindowName(foreground="a0a0a0",),
                widget.Notify(),
                widget.Systray(),
                widget.KeyboardLayout(configured_keyboards=["us", "ru"]),
                widget.Volume(foreground="70ff70",),
                widget.BitcoinTicker(currency="rub", format="{buy}"),
                widget.Clock(foreground="a0a0a0",
                             fmt="%H:%M %d.%m.%Y",),
            ],
            18,
        ),
    ),
]
{{< /highlight >}}
эти хуки необходимы для того что бы некоторые всплывающие окна и диалоги автоматом переключались в режим `floating` и не открывались во весь экран  
{{< highlight python >}}
@hook.subscribe.client_new
def dialogs(window):
    if (window.window.get_wm_type() == 'dialog'
            or window.window.get_wm_transient_for()):
        window.floating = True


@hook.subscribe.client_new
def vue_tools(window):
    if((window.window.get_wm_class() == (
        'sun-awt-X11-XWindowPeer', 'tufts-vue-VUE')
            and window.window.get_wm_hints()['window_group'] != 0)
            or (window.window.get_wm_class() == (
                'sun-awt-X11-XDialogPeer', 'tufts-vue-VUE'))):
        window.floating = True


@hook.subscribe.client_new
def idle_dialogues(window):
    if((window.window.get_name() == 'Search Dialog') or
      (window.window.get_name() == 'Module') or
      (window.window.get_name() == 'Goto') or
      (window.window.get_name() == 'IDLE Preferences')):
        window.floating = True


@hook.subscribe.client_new
def libreoffice_dialogues(window):
    if ((window.window.get_wm_class() == (
        'VCLSalFrame', 'libreoffice-calc')) or
            (window.window.get_wm_class() == (
                'VCLSalFrame', 'LibreOffice 3.4'))):
        window.floating = True
{{< /highlight >}}
а здесь мы создаем метод для запуск программ и описываем программы которые необходимо запускать при старте `qtile`  
{{< highlight python >}}
def is_running(process):
    s = subprocess.Popen(["ps", "axw"], stdout=subprocess.PIPE)
    for x in s.stdout:
        if re.search(process, x):
            return True
    return False


def execute_once(process):
    if not is_running(process):
        return subprocess.Popen(process.split())


@hook.subscribe.startup
def startup():
    execute_once("nm-applet")
    execute_once("synergy")
    execute_once("feh --bg-fill /home/pavel/Pictures/arch.png")
    execute_once(
        'setxkbmap -layout us,ru -option grp:caps_toggle,grp_led:scroll')
{{< /highlight >}}
