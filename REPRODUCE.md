# Портфолио Daria Bodaikina — спецификация для воспроизведения

Этот файл — самодостаточное ТЗ. Отдай его нейросети (Claude, Cursor, любой ассистент)
вместе с папкой `assets/` — и она соберёт страницу с нуля, без доступа к исходному аккаунту.

**Промпт для нейронки:** «Воспроизведи страницу строго по этой спецификации.
Один HTML-файл, стили только инлайновые (кроме блока в `<head>`, описанного ниже).
Картинки бери из папки assets. Ничего не добавляй от себя.»

---

## 0. Что лежит рядом

| Путь | Что |
|---|---|
| `portfolio-daria-bodaikina.html` | Готовая офлайн-версия. Открывается двойным кликом в любом браузере, всё внутри (шрифты, картинки, скрипты). Не редактировать вручную. |
| `assets/` | Исходные картинки: `showreel.gif`, `case-crossborder.png`, `whale-body.png`, `whale-tail.png`, `whale-full.png` (кит целиком, референс). |
| `source/` | Исходник в формате Design Component: `Studio Layout.dc.html` + рантайм `support.js`, `image-slot.js` и сайдкар `.image-slots.state.json` (в нём лежат залитые в слоты картинки правой панели, base64). Нужен, только если продолжаешь работу в том же окружении. |
| `REPRODUCE.md` | Этот файл. |

Если нужен просто рабочий сайт — берётся `portfolio-daria-bodaikina.html`.
Если нужно пересобрать с нуля в другом инструменте — читается спецификация ниже.

---

## 1. Общая идея

Одноэкранная портфолио-страница, две колонки на всю высоту окна, без вертикального
скролла у страницы целиком:

- **Слева** — единственная скроллящаяся колонка с контентом (`overflow-y:auto`).
- **Справа** — статичная панель фиксированной ширины `clamp(340px, 44vw, 660px)`,
  никогда не скроллится. Внутри неё две подколонки: основная (имя + большое фото + текст)
  и узкий «архив» из трёх маленьких картинок, `flex:0 0 clamp(120px,13vw,208px)`.
- Курсор мыши заменён на **оригами-кита** из двух PNG-слоёв: тело + хвост,
  хвост машет тем сильнее, чем быстрее движется мышь.

Внешняя рамка: весь макет — светлый лист `#f2efe9` внутри оранжевой рамки
`border:6px solid #d0491a`, `position:fixed; inset:0; display:flex; overflow:hidden`.

---

## 2. Дизайн-язык (жёстко)

**Цвета**

| Токен | Значение | Где |
|---|---|---|
| Бумага | `#f2efe9` | фон листа |
| Оранжевый | `#d0491a` | рамка, фон вокруг, акцентный span в PRINCIPLES |
| Чернила | `#26261f` | заголовки, метки секций, акцентный текст |
| Вторичный | `#3a3a38` | футер, вторичные метки (CONTEXT / CONSTRAINTS…) |
| Базовый текст | `#111318` | наследуется от контейнера |
| Разделители | `1px solid rgba(0,0,0,.35)` | между секциями; `rgba(0,0,0,.28)` — вертикальная линия слева от правой панели |
| Скроллбар | thumb `#8a8a86`, трек прозрачный, ширина 8px, radius 8px |

**Шрифты** (Google Fonts, подключить в `<head>`):
`Archivo` 700 + 700 italic — заголовки; `Cutive Mono` — весь остальной текст, включая
крупный текст блока PRINCIPLES.

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Archivo:ital,wght@0,700;1,700&family=Cutive+Mono&display=swap" rel="stylesheet">
```

**Типографика**

- H1 hero: Archivo 700, `clamp(38px,6.4vw,132px)`, `line-height:.92`, `letter-spacing:-.01em`.
- Надпись `About` над hero: Archivo 700 **italic**, 16px, `#26261f`.
- Метки секций в левой колонке: 11px, `letter-spacing:.08em`, UPPERCASE, `#26261f`
  (вторичные — `#3a3a38`).
- Заголовок кейса: Archivo 700, `clamp(22px,2.2vw,34px)`, `line-height:1.05`.
- Крупный текст PRINCIPLES: Cutive Mono **700**, `clamp(24px,2.6vw,40px)`, `line-height:1.15`.
- Абзацы: `clamp(13px,1.05vw,15.5px)` — `clamp(14px,1.15vw,17px)`, `line-height:1.45–1.5`,
  `max-width:820px`.
- Годы в биографии: Archivo 700, `clamp(15px,1.3vw,19px)`.
- Футер: 12px, `letter-spacing:.04em`, `#3a3a38`.

**Сетка и отступы**

- Каждая секция левой колонки — `display:grid`,
  `grid-template-columns: clamp(80px,9vw,120px) 1fr`, `gap: clamp(20px,3vw,44px)`
  (в блоке кейса ещё и построчный `gap: clamp(22px,2.4vw,34px)`).
- Паддинги секций: `clamp(22px,2.4vw,32px) clamp(28px,4vw,56px) clamp(28px,3vw,44px)`.
- Все размеры на `clamp()` — брейкпоинтов нет, страница тянется непрерывно.
- Группы элементов — flex/grid + `gap`, никогда не margin по одному.

---

## 3. Структура левой колонки (сверху вниз)

1. **hero** — `min-height:100vh`, `flex-direction:column; justify-content:space-between`.
   Сверху italic `About`, снизу H1 «Senior product designer».
2. **SHOWREEL** — `assets/showreel.gif` на всю ширину, `aspect-ratio:4/3`, `object-fit:cover`;
   под ним абзац: «Most of my recent work is under NDA, so I use selected older cases not as a
   gallery of current UI trends, but as evidence of how I think…»
3. **PRINCIPLES** — один крупный абзац. Средняя фраза («With AI, shorter product bets, and faster
   engineering cycles, design has to stay closer to execution.») обёрнута в
   `<span style="color:#d0491a">` — единственный цветной акцент в тексте.
4. **ONE CASE** — «Cross-Border Invoicing Flow»:
   заголовок → картинка `assets/case-crossborder.png` (`aspect-ratio:1871/1397`,
   `object-fit:cover`, `border-radius:14px`) → далее пары «метка / контент» той же сеткой:
   CONTEXT (абзац), CONSTRAINTS (ul, 3 пункта), DECISION (2 абзаца), OUTCOME (ul, 5 пунктов),
   IMPACT (один абзац крупнее, `#26261f`). Списки: `padding-left:1.1em`, `gap:5px`.
   Нижний паддинг секции больше: `clamp(60px,6vw,110px)`.
5. **BIOGRAPHY** — 5 блоков вида «годы — роль» + абзац:
   2025–Now / AI Product Designer · 2021–2025 / Senior-Staff Product Designer ·
   2016–2021 / Product Designer · 2014–2016 / Communication Designer → IT ·
   Before 2014 / Art → Design. Строка периода: годы, тире `opacity:.45`, роль — одна flex-строка
   с `gap:0 10px`, `align-items:baseline`, `flex-wrap:wrap`.
6. **footer** — слева «DARIA BODAIKINA», справа две ссылки:
   `mailto:d.bodaikina@gmail.com` («D.BODAIKINA@GMAIL.COM») и
   `https://t.me/bodaikina` («TELEGRAM: @BODAIKINA»). Цвет ссылок `#3a3a38`,
   обязательно задать `a` и `a:hover` явно, чтобы не появился браузерный синий.

---

## 4. Правая панель

Основная подколонка (`flex:1 1 auto`, паддинги `clamp(16px,1.6vw,22px) clamp(18px,1.8vw,26px)`,
`gap:clamp(10px,1.4vw,16px)`):

1. Имя в две строки — «Daria» / «Bodaikina», `clamp(22px,2.2vw,30px)`, Cutive Mono.
2. **Большое фото** — растягивается на всю оставшуюся высоту (`flex:1 1 0; min-height:0`),
   `object-fit:cover`, `object-position:50% 0%` (прижато к верху), `border-radius:2px`.
3. Строка 13px: «Creative AI Product Designer, Staff-level Product Designer with 8+ years in teams».
4. Абзац 13.5px, `line-height:1.45`, `#26261f`: про работу со сложными data-heavy интерфейсами —
   банковские системы, транзакционные и approval-флоу, multi-role B2B; на стыке архитектуры,
   UX и продуктовой стратегии.

Узкая подколонка «архив»: три картинки одна под другой, каждая `flex:1 1 0; min-height:0`,
`border-radius:2px`, `gap:8px`, паддинг `clamp(14px,1.4vw,20px) 8px 8px`.
Первая — `object-fit:contain` на фоне `#edeff3`, две другие — `cover`.

В исходнике это компоненты `<image-slot>` (drag&drop плейсхолдеры). При воспроизведении
без этого рантайма — просто `<img>` с теми же `object-fit` / `object-position` / радиусами;
сами файлы вытаскиваются из `source/.image-slots.state.json` (ключи `featured`,
`arch1`, `arch2`, `arch3`, значение `u` — data-URL WebP) или заливаются заново.

---

## 5. Курсор-кит

Разметка: `<div id="om-whale">` — `position:fixed; top:0; left:0; z-index:9999;
pointer-events:none`. Внутри обёртка `76×61px` с `filter:drop-shadow(0 3px 5px rgba(20,40,55,.28))`,
внутри неё два абсолютных `<img>`: сначала **хвост** (`transform-origin:66.3% 35.5%` — точка
крепления к телу), поверх — **тело**.

CSS (единственное, что нельзя инлайном):

```css
@media (hover:hover) and (pointer:fine){ *{cursor:none !important} }
html[data-whale-off] *{cursor:auto !important}
html[data-whale-off] #om-whale{display:none !important}
@media (hover:none),(pointer:coarse){ #om-whale{display:none !important} }
```

Логика в rAF-цикле:

- `pointermove` пишет цель `tx/ty`; позиция догоняет её лерпом `0.17 * dt` (`dt` в кадрах 60fps).
- Скорость `speed = hypot(vx,vy)` → сглаженная переменная `wag`; фаза
  `phase += (0.09 + wag*0.34) * dt * energy`, амплитуда `amp = (3 + wag*17) * energy`.
  Хвост: `rotate(sin(phase) * amp)`.
- Направление: при `|vx| > 0.4` запоминается знак; кит зеркалится `scaleX(±1)`,
  наклоняется `tilt = clamp(vy*1.4, -14, 14) * flip`.
- Точка привязки к курсору: `hotspotX = size * (dir === 1 ? 0.97 : 0.03)`,
  по вертикали `y - size*0.268`. Высота обёртки `= size * 0.798`.
- Всё через `element.style.transform`, не через ререндер; `will-change:transform`.

Три настраиваемых параметра (в исходнике — props/Tweaks):
`whaleCursor` (boolean, `true`), `whaleSize` (40–160px, `76`), `tailEnergy` (0.3–2.2, `1.5`).

⚠️ **Про редактирование:** кит всегда находится строго под указателем, поэтому визуальный
редактор хиттестит его, а не элементы под ним. Перед правкой макета выключай тумблер
`whaleCursor` (он ставит `data-whale-off` на `<html>`), потом включай обратно.

---

## 6. Правила, которые важно не потерять

- Стили **только инлайновые**. В `<head><style>` — исключительно ресеты, скроллбар и
  media-запросы курсора; никаких классов и дизайн-токенов в CSS.
- Скроллится **только** левая колонка. `html,body{height:100%;margin:0;overflow:hidden}`.
- Максимальная ширина текстовых блоков — 820–900px, иначе строки разъезжаются на широких экранах.
- Никаких градиентов, теней у карточек, эмодзи и цветных плашек: язык — бумага, моно-шрифт,
  один оранжевый акцент.
- Vibe Design System привязана к проекту, но макет её **не использует** — это осознанное решение
  (собственная типографика и палитра). Перевод на vibe = отдельный редизайн, не мелкая правка.
