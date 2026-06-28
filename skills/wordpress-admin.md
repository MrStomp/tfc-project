# WordPress Admin — стандарт настройки
> Свод правил по организации WordPress-админки для клиентских сайтов.
> Основан на опыте проекта TFC (tfc.kz), Chats 8–8.2.
> Плагин: SCF (Secure Custom Fields, fork ACF).

---

## 1. Структура меню админки

**Принцип:** клиент не должен видеть ничего лишнего. Только свой контент.

### Что скрывать:
- SCF / ACF (плагин)
- Настройки WordPress
- Плагины
- Медиафайлы из бокового меню (только через кнопку внизу)
- Инструменты, Комментарии

### Что оставлять:
- Единый раздел **Страницы** с кастомным списком страниц
- Медиатека — отдельная кнопка внизу списка (через разделитель)

### Реализация (functions.php):
```php
// Скрыть стандартные пункты меню
add_action('admin_menu', function() {
    remove_menu_page('upload.php');         // Медиатека из меню
    remove_menu_page('options-general.php'); // Настройки
    remove_menu_page('plugins.php');         // Плагины
    // ... и т.д.
});

// Редирект edit.php?post_type=page → кастомный список
add_action('admin_init', function() {
    global $pagenow;
    if ($pagenow === 'edit.php' && isset($_GET['post_type']) && $_GET['post_type'] === 'page') {
        wp_redirect(admin_url('admin.php?page=tfc-pages'));
        exit;
    }
});
```

### WP-логотип в админке:
- Ведёт на кастомный список страниц (не на dashboard)
- Логотип в Gutenberg — тоже туда

---

## 2. Организация ACF/SCF полей

### Принцип группировки:
- Одна группа = одна секция страницы
- Группа прикреплена к конкретной странице по ID (`post_id = $_front_id`)
- Никаких глобальных полей без явной привязки

### Типы полей:
| Что | Тип SCF |
|-----|---------|
| Текст заголовка | `text` |
| Длинный текст / описание | `textarea` |
| Ссылка / URL | `url` |
| Изображение | `image` (return: url) |
| Повторяющийся блок | `repeater` |
| Число / статистика | `number` |
| Булево | `true_false` |

### Обязательно для каждого поля:
- `instructions` — подсказка для контентщика (размер фото, максимум символов)
- `default_value` — предзаполнение, чтобы сайт не был пустым после установки

---

## 3. Options Page (для глобальных данных)

Использовать для данных которые повторяются на всех страницах:
- Футер (адреса офисов, соцсети, навигация, политика конфиденциальности)
- Контакты (телефон, email)
- Глобальные настройки

```php
if (function_exists('acf_add_options_page')) {
    acf_add_options_page([
        'page_title' => 'Настройки сайта',
        'menu_title' => 'Настройки',
        'menu_slug'  => 'site-settings',
        'capability' => 'manage_options',
    ]);
    acf_add_options_sub_page([
        'page_title'  => 'Футер',
        'menu_title'  => 'Футер',
        'parent_slug' => 'site-settings',
    ]);
}
```

**Важно:** Options Page не доступна из списка страниц — только через подменю в сайдбаре. Предупредить клиента.

---

## 4. Получение полей в PHP-шаблонах

Всегда явно передавать `$_front_id` — не полагаться на глобальный `$post`:

```php
$_front_id = get_option('page_on_front');

// Обычные поля
$title = get_field('hero_title', $_front_id);

// Repeater
$items = get_field('reviews', $_front_id);
if ($items): foreach ($items as $item): ?>
    <div><?= esc_html($item['text']) ?></div>
<?php endforeach; endif;

// Options Page
$offices = get_field('footer_offices', 'option');
```

---

## 5. Предзаполнение при активации (prefill)

Чтобы сайт сразу работал с демо-контентом:

```php
// functions.php
add_action('acf/init', function() {
    if (get_option('tfc_acf_prefilled')) return;
    
    $front_id = get_option('page_on_front');
    if (!$front_id) return;
    
    // Hero
    update_field('showreel_url', 'https://youtu.be/XXXXXXXXX', $front_id);
    
    // Stats
    update_field('stat_1', '4', $front_id);
    // ...
    
    // Reviews (repeater)
    update_field('reviews', [
        ['name' => 'Клиент 1', 'role' => 'CEO', 'text' => 'Текст отзыва...'],
    ], $front_id);
    
    update_option('tfc_acf_prefilled', true);
});
```

---

## 6. Подсказки для контентщика

### Изображения:
- Всегда указывать размер в `instructions`: «425×314px, PNG с прозрачным фоном»
- Alt-текст: «alt-текст для SEO»

### Текстовые поля:
- Для отзывов/описаний: живой счётчик символов
- Цвет индикатора: серый > 30 остаток, оранжевый < 30, красный при превышении

### Repeater:
- `button_label` — понятная надпись («Добавить фото», «Добавить отзыв»)
- Если есть ограничение — `max` на repeater
- Подсказка о порядке элементов если важен

---

## 7. Табы-разделители в редакторе страниц

Для длинных страниц с множеством секций — добавлять вкладки-разделители:

```php
// SCF: message field с типом "tab"
// Пример: Hero / Clients / About / Stats / Why Us / Reviews
```

Делать в начале работы над админкой, не в конце — потом сложнее перегруппировать поля.

---

## 8. Типичные ошибки (антипаттерны)

| Ошибка | Правильно |
|--------|-----------|
| `get_field()` без `$_front_id` | Всегда передавать явный ID |
| Хардкод данных в PHP | ACF поле + fallback на хардкод |
| Одна большая группа полей | Группа на секцию |
| Нет instructions на полях | Инструкция для каждого поля |
| Нет prefill | Демо-данные при первом запуске |
| Конфликт имён полей | Проверять существующие поля перед регистрацией |
| `position` вместо `role` в reviews | Называть поля по смыслу, не по шаблону |

---

## 9. Showreel / видео

- YouTube URL → iframe при клике на превью (не автозапуск)
- Кнопка play поверх превью — всегда
- ACF поле `showreel_url` типа `url`
- В PHP: извлекать video ID через regex, строить iframe динамически

---

## 10. Чеклист перед сдачей проекта

- [ ] Все секции подключены к ACF полям
- [ ] Есть prefill с демо-данными
- [ ] Меню админки очищено
- [ ] Каждое поле имеет instructions
- [ ] Options Page для футера работает
- [ ] Табы-разделители в редакторе главной
- [ ] Медиатека доступна через кнопку внизу
- [ ] WP-логотип ведёт на кастомный список страниц
