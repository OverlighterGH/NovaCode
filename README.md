<p align="center">
  <img src="nova-code.svg" alt="Nova Code" width="720">
</p>

<h1 align="center">Nova Code</h1>

<p align="center">
  Анімований ASCII-логотип для терміналу та браузера.<br>
  Літери з'являються по одній, всередині них букви й цифри, потім усе заливається кольором <code>#5F2FD4</code>.
</p>

---

## Що всередині

| Файл | Для чого |
| --- | --- |
| `nova_code.py` | анімація в терміналі (Windows CMD, Windows Terminal, Linux, macOS) |
| `nova_code.html` | та сама анімація в браузері, прозорий фон, клік повторює |
| `nova-code.svg` | анімована картинка для цього README (працює прямо на GitHub) |
| `make_svg.py` | генерує `nova-code.svg` заново, якщо змінити колір або напис |

## Запуск у терміналі

Потрібен лише Python 3, нічого встановлювати не треба.

```bash
python nova_code.py
```

На Windows краще запускати у Windows Terminal або в CMD на Windows 10 і новіше (потрібна підтримка ANSI-кольорів).
Вікно скрипт розтягує сам до 80x24.

## Запуск у браузері

Відкрий `nova_code.html` подвійним кліком. Клік по сторінці запускає анімацію ще раз.

## Свій колір

Основний колір `#5F2FD4` задається у трьох місцях:

- `nova_code.py`: числа `95, 47, 212` у функції `frame`
- `nova_code.html`: ті самі числа `95`, `47`, `212` у функції `draw`
- `make_svg.py`: змінна `MAIN` на початку файлу, потім запусти `python make_svg.py`

## Як додати на GitHub

1. Поклади `README.md` і `nova-code.svg` в корінь репозиторію (в одну папку).
2. Зроби commit і push, анімація з'явиться вгорі README.

```bash
git add README.md nova-code.svg nova_code.py nova_code.html make_svg.py
git commit -m "Nova Code logo"
git push
```

> GitHub не виконує JavaScript у README, тому для нього зроблено окремий анімований `nova-code.svg`.
> Він підключається звичайним тегом `<img>` і програється сам.
