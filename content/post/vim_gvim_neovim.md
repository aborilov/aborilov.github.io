---
title: "Сравнение производительности vim, gvim и neovim"
date: 2015-04-22T15:00:00+03:00
---

Появилась задача проализировать 1.5млн строк логов, конечно `vim` в этом очень помог, но запуск макросов на таком количестве строк
занимал ощутимое время и поэтому возник вопрос оптимизации.  
Написав простой макрос и запустив его на 1000 строк я получил следующие результаты:  
`gvim` на 15% **медленее** `vim`, а `nvim` на 25% **быстрее** `vim`.
