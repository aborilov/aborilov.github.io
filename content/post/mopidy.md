---
title: "Плеер mopidy"
date: 2014-06-03T15:00:00+03:00
---

Плеер [Mopidy](http://www.mopidy.com/) оказался для меня открытием.
Плеер может работать как сервер MPD. Управляться клиентом MPD.
Проигрывать как локальный контент, так и с SoundCloud, GooglePlay Music, TuneIn ... etc.  
Если запускаем плеер как сервис, то конфиг лежит в `/etc/mopidy/mopidy.conf`  
```
[logging]
config_file = /etc/mopidy/logging.conf
debug_file = /var/log/mopidy/mopidy-debug.log

[local]
enabled = true
data_dir = /var/lib/mopidy/local
media_dir = /var/lib/mopidy/media
playlists_dir = /var/lib/mopidy/playlists

[soundcloud]
enabled = true
explore_songs = 25
auth_token = ####

[tunein]
timeout = 5000

[mpd]
enabled = true
hostname = 127.0.0.1
port = 6600
#password =
#max_connections = 20
#connection_timeout = 60
#zeroconf = Mopidy MPD server on $hostname
```
Я активировал tunein, mpd и soundcloud. [tunein](https://github.com/kingosticks/mopidy-tunein) и [soundcloud](https://github.com/mopidy/mopidy-soundcloud) доставляются отдельными плагинами. Mpd встроен изначально.
Текущая конфигурация смотрится командой

`mopidy config`
