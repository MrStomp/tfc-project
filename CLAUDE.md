# TFC — tfc.kz

Основной CLAUDE.md находится внутри темы:
`app\public\wp-content\themes\tfc\CLAUDE.md`

---

## Stack
WordPress custom theme (PHP + HTML/CSS/JS). Hosting: SpaceWeb.
Grid: 1920px, content zone 1760px, padding 80px, 12 columns.
Font: Onest (Google Fonts).

## Пути
- Тема: `app\public\wp-content\themes\tfc\`
- CSS: `app\public\wp-content\themes\tfc\assets\css\main.css`
- JS: `app\public\wp-content\themes\tfc\assets\js\main.js`
- Ассеты: `app\public\wp-content\themes\tfc\assets\img\`
- Секции: `app\public\wp-content\themes\tfc\sections\`

## Design system
- Background: #F6F6F6
- Black: #181A1D
- Accent: #C0EF24
- Font: Onest

## Figma
File key: FDCPZi42bF9EEuDwTtwBXy
Page: Page 1 (home_25_05)

## Figma node map (home_25_05: 2448:190)
- header: 2448:471
- HERO: 2448:440
- CLIENTS: 2448:385
- CLIENTS ALL: 2448:2
- ABOUT: 2448:373
- STATS: 2448:336
- WHY US: 2448:296
- CASES: 2448:278
- PROCESS: 2448:244
- REVIEWS: 2448:229
- CTA: 2448:220
- FOOTER: 2448:197

## Status
- [x] Chat 0 — base theme setup
- [x] Chat 1 — Header
- [x] Chat 2 — Hero
- [x] Chat 3 — Clients + About
- [x] Chat 4 — Stats + Why Us
- [x] Chat 5 — Cases + Process
- [x] Chat 6 — Reviews + CTA + Footer
- [x] Chat 7 — Animations
- [x] Chat 8 — ACF Admin
- [ ] Chat 9 — Pages (SMM, Marketing, Branding, Production)

## Section Y-coordinates
| Секция  | Figma Y_start | Height | Canvas formula   |
|---------|---------------|--------|------------------|
| HERO    | 108           | 1819   | element_Y - 108  |
| CLIENTS | 1927          | 762    | element_Y - 1927 |
| ABOUT   | 2689          | 1265   | element_Y - 2689 |
| STATS   | 4049          | 1253   | element_Y - 4049 |
| WHY US  | 5302          | 1238   | element_Y - 5302 |
| CASES   | 6540          | 1220   | element_Y - 6540 |
| PROCESS | 7760          | 1716   | element_Y - 7760 |
| REVIEWS | TBD           | TBD    | element_Y - TBD  |
| CTA     | TBD           | TBD    | element_Y - TBD  |
| FOOTER  | TBD           | TBD    | element_Y - TBD  |

## Запреты
- НИКОГДА не читать `tfc_wireframes.html` и `tfc_wireframes_26_06.html` — тяжёлые файлы для клиента, не нужны в работе

## Rules
- Перед кодингом: get_design_context(section_node) → Y_start и H в таблицу выше
- Все элементы — position:absolute в .canvas-inner, без wrapper-div
- Писать расчёт в CSS-комментарии: /* Figma Y=4144 - 4049 = 95px */
- Одно изменение за раз: координаты и canvas height — разными редактами

## Сепараторы
- Принадлежат секции которая ЗАКАНЧИВАЕТСЯ — в начало секции не добавлять
- Перед добавлением: проверить метадату предыдущей секции на наличие separator--bottom

## Ассеты — PNG vs CSS
ЗАПРЕЩЕНО качать как PNG: ellipse/circle, rectangle, line — любая простая геометрия.
Это всегда CSS, даже если это отдельный слой в Figma. PNG с Figma экспортируется с белым фоном.

Качать как PNG только: star, blob, union, splash (сложный вектор), фото, underline-декорации.
Критерий перед download_assets: "описывается одним CSS-свойством?" → если да, не качать.

## Текстовые переносы
- Никогда не добавлять br вручную — width из Figma обеспечивает перенос
- Исключение: текст явно разбит на строки с разными y-координатами в метадате
