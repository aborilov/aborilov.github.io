---
title: "Выбор и создание сессий tmux"
date: 2015-06-17T14:15:00+03:00
---

С помощью этой команды можно подключиться к уже существующей сессии tmux или создать новую сессию с заданным именем.
{{< highlight sh >}}
sh -c "tmux ls -f '#{session_name}' | dmenu -l 7 | xargs -i{} urxvtcd -e sh -c 'tmux attach -t {} || tmux new -s {}'"
{{< /highlight >}}

Выполнение данной команды можно повесить на комбинацию клавишь.
Для работы необходимо приложение dmenu.
Терминал можно заменить на любой другой.
