# Figma → Pixel-Perfect вёрстка
> Универсальный свод правил для любого проекта с pixel-perfect вёрсткой по Figma-макету.
> Основан на опыте проекта TFC (tfc.kz), Chats 1–8.

---

## 0. Рабочий процесс и лимиты

**Сначала ТЗ — потом работа.**
- Перед любой задачей: обсудить план с пользователем и получить подтверждение
- Не запускать тяжёлые операции (чтение больших файлов, поиск по всем чатам) без предупреждения
- Если операция может съесть много токенов — предупредить и предложить более дешёвую альтернативу
- Если непонятно как делать → спросить, не угадывать

**Одно изменение за раз.**
- Координаты и canvas height — разными редактами
- Если что-то сломалось — сначала понять ПОЧЕМУ, потом менять
- Не делать "быстрый фикс" который может отменить предыдущий правильный

---

## 1. Перед началом каждой секции

1. `get_design_context` на NODE САМОЙ СЕКЦИИ (не дочерние) → получить `frame height` и `Y_start`
2. `get_design_context` на родительский фрейм страницы → подтвердить Y-координату секции
3. Записать в CLAUDE.md таблицу:
   ```
   SECTION: Y_start=..., H=...
   ```
4. Скачать все ассеты ЗАРАНЕЕ — до начала кодинга

---

## 2. Система координат

```
canvas_top = figma_element_Y - figma_section_Y_start
left       = figma_element_X
```

- Все элементы — `position: absolute` прямо в `.canvas-inner`, без лишних wrapper-div
- Писать расчёт в CSS-комментарии: `/* Figma Y=2108 - 1927 = 181px */`
- Никогда не менять canvas height и координаты элементов в одном редакте
- Никогда не угадывать координаты — только из `get_design_context`

---

## 3. Ассеты — что скачивать, что делать CSS

### Алгоритм (применять по порядку):
1. В design context есть `<img src={imgXxx}>` → скачать (Figma уже решил)
2. Тип ноды: `Line` / `Rectangle` / `Ellipse` → CSS (border-radius, background, height:1px)
3. Размер экспорта >5KB → почти всегда скачать
4. "Описывается одним CSS-свойством?" → да=CSS, нет=скачать

### Запрещено скачивать (делать CSS):
- `ellipse` / `circle` → `border-radius: 50%`
- `rectangle` → `background + border-radius`
- `line` (прямая) → `height: 1-2px + background`

### Всегда скачивать:
- icon, arrow, button-icon (несколько векторных путей)
- underline-decoration с органической формой (волна, кривая)
- star, blob, union, splash, Group с несколькими слоями
- фото, GIF

### ГЛАВНОЕ ПРАВИЛО (нарушалось много раз):
**PNG из Figma = белый фон. SVG из Figma = содержит фоновые rect. Оба требуют очистки.**

**Алгоритм после download_assets:**
```
1. Скачать как SVG (defaultFormat: "svg")
2. Сразу очистить фоновые rect — PowerShell:

$content = Get-Content $path -Raw -Encoding UTF8
$content = $content -replace '<rect width="\d+" height="\d+" fill="#F[56]F[56]F[56]"/>\s*', ''
$content = $content -replace '<rect width="\d+" height="\d+" transform="[^"]*" fill="#F[56]F[56]F[56]"/>\s*', ''
Set-Content $path $content -Encoding UTF8 -NoNewline

3. Проверить — не должно быть fill="#F5F5F5" или fill="#F6F6F6" в rect-элементах
```

Исключение: растровые фото/GIF скачивать без изменений (PNG, scale:2).

### Если download_assets вернул rawImages:
Брать URL из `rawImages[0].url` и качать через PowerShell `Invoke-WebRequest`.

---

## 4. Маски в Figma

Если элемент обрезан маской — экспортированный PNG будет меньше Figma-размера.

**Алгоритм:**
1. Проверить в Figma: есть ли маска на элементе или родителе
2. Если есть: `сдвиг = original_size - cropped_size` в сторону обрезки
3. CSS-координата: `left = figma_X + сдвиг` (если обрезан слева — left сдвинуть вправо)

Пример: Figma left=-45px, PNG обрезан на 45px слева → CSS left=0px.

---

## 5. Сепараторы

- Сепаратор принадлежит секции которая **заканчивается** — добавлять как `separator--bottom`
- Никогда не добавлять сепаратор в начало следующей секции
- Перед кодингом: проверить метадату предыдущей секции — если есть `separator--bottom`, top-separator текущей не нужен

---

## 6. Текстовые переносы

- **Никогда** не добавлять `<br>` вручную
- Ширина элемента из Figma (`width: Xpx` на элементе) обеспечивает правильный перенос
- Исключение: только если в Figma текст явно разбит на строки с разными y-координатами в метадате

---

## 7. Анимации появления (система .anim)

```css
.anim {
  opacity: 0;
  transform: translateY(50px);
  transition: opacity 1.5s ease, transform 1.5s ease;
}
.anim.is-visible {
  opacity: 1;
  transform: translateY(0);
}
```

IntersectionObserver на `.canvas-wrap` → добавляет `.is-visible` всем `.anim` внутри с задержкой `index * 100ms`.

### Исключения (конфликты transform):
- `.hero-canvas .anim` — только opacity, без translateY
- `.clients-slider-wrap.anim`, `.reviews-swiper.anim` — без translateY (конфликт Swiper)
- `.about-underlining--2.anim` — `rotate(180deg) translateY(-50px)` (ось перевёрнута)
- `.cta-bg`, `.footer-bg` — без .anim вообще
- Элементы с `translateX(-50%)` (по центру): `translateX(-50%) translateY(50px)`

---

## 8. Параллакс

Механика: `offset = scrollY / ratio` (ratio = viewport_width / 1920).

Коэффициенты Hero (tfc.kz — для справки):
- planet: 0.55, ornament: 0.42, pyramid: 0.28
- eyes/showreel: 0.4 / 0.1 (относительно центра видео в viewport)

Для нового проекта — подбирать визуально, начинать с 0.2–0.4.

---

## 9. Слайдеры (Swiper)

Clients: `slidesPerView:4, spaceBetween:20, loop:true, speed:600, autoplay:1500ms`
Reviews: `slidesPerView:1, loop:true, speed:1200, touchRatio:1/ratio, threshold:5, autoplay:5000ms`

`overflow: visible` на `.swiper` — для выхода слайдов за края canvas.

---

## 10. Частые системные ошибки (антипаттерны)

| Ошибка | Правильно |
|--------|-----------|
| Угадывать координаты | get_design_context → точные значения |
| PNG с Figma без очистки | SVG → очистить rect → поставить |
| CSS вместо ассета из Figma | Скачать → img по координатам |
| Менять canvas height и координаты вместе | Разными редактами |
| Добавлять `<br>` для переноса | Ширина из Figma делает перенос |
| Добавлять сепаратор в начало секции | separator--bottom в конце предыдущей |
| Wrapper-div вокруг элементов | position:absolute прямо в .canvas-inner |
| Читать координаты из screenshot | Только из get_design_context / metadata |

---

## 11. Оптимизация — предложения для новых проектов

### Независимость от дизайнера
- **Design tokens сразу**: цвета, шрифты, отступы — зафиксировать в CSS переменных с первого чата
- **Компонентный подход**: повторяющиеся элементы (карточки, кнопки) — PHP-компоненты с параметрами, не копипаст
- **Референс-библиотека**: собирать типовые секции (hero, clients, reviews, cta) — потом адаптировать, не делать с нуля

### Снижение глюков
- CLAUDE.md обновлять в конце каждого чата — не копить долг
- Memory писать сразу когда найдено нестандартное решение
- Перед стартом чата: прочитать CLAUDE.md + свежий memory → не повторять старые ошибки
- Сложные секции (маски, анимации, слайдеры) — сначала статика, потом анимация отдельным чатом
