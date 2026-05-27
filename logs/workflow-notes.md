# Workflow Notes — HW06 Multiprompt Tests

## Базовые параметры
- **Модель:** sd_xl_turbo_1.0_fp16  
- **Workflow:** w06_multiprompt (3 prompt-слоя + ConditioningCombine + ConditioningAverage)  
- **Seed (базовый):** 299880580668615  
- **Aspect Ratio:** 1024×576 (строгий top‑down 16:9)  
- **Sampler / Scheduler:** euler_ancestral / karras  
- **Steps:** 20  
- **CFG:** 5.0  
- **Фиксированные параметры:** ракурс top‑down, индустриальный стиль, silhouette‑priority  

## Prompt-блоки
- **constants / identity:** силуэт Кайла, массивные плечи, ранец, янтарный визор  
- **style / world:** industrial sci-fi brutalism, warm amber light / electric blue + neon purple  
- **modifier / state:** emotion / light / glitch‑intensity / shot‑distance  

---

# Таблица тестов

| Вариант | Seed            | Weight | Что меняли                    | Что изменилось                                 | Keep / Drop |
| ------- | --------------- | ------ | ----------------------------- | ---------------------------------------------- | ----------- |
| 01      | 299880580668615 | 0.3    | Emotion (neutral→focused)     | Лёгкое изменение, визор ярче                   | Drop        |
| 02      | 299880580668615 | 0.6    | Emotion (focused→aggressive)  | Плечи сузились, механический дизайн            | Drop        |
| 03      | 299880580668615 | 0.4    | Shot distance (mid→close)     | Хорошая читаемость визора, silhouette стабилен | Keep        |
| 04      | 299880580668615 | 0.5    | Light (amber→low amber)       | Сильнее ощущается индустриальный baseline      | Keep        |
| 05      | 299880580668615 | 0.7    | Costume detail (extra cables) | Лишние элементы ломают чистоту силуэта         | Drop        |
| 06      | 299880580668615 | 0.45   | State (glitch‑intensity)      | silhouette сохраняется                         | Keep        |

---

# Выводы по тестам

### 1. Рабочий диапазон весов
**0.35–0.5** — оптимальный диапазон modifier.  
Ниже 0.3 — эффект почти не читается.  
Выше 0.6 — silhouette начинает разрушаться.

### 2. Что сильнее всего влияет на узнаваемость
- форма плеч и ранца;  
- дугообразный янтарный визор;  
- пропорции torso сверху;  
- отсутствие лишних аксессуаров.

### 3. Что ломает consistency
- чрезмерный glitch (>0.65);  
- изменение формы шлема;  
- добавление крупных элементов костюма;  
- попытка сделать эмоции слишком «человеческими».

### 4. Baseline для дальнейшей работы
- Weight = **0.4–0.45**  
- Seed = **299880580668615**  
- Light = **soft amber**  
- Shot = **mid top‑down**  
- Glitch = **0.45** для зеркальной формы  

## Шаг 1. Img2img test — стабилизация исходного кадра

**Контекст:**
Исходный кадр — инженерный костюм (02-w06-hero-emotion.png).  
Проблема: плоский свет и слабая читаемость формы.  
Цель: усилить объём и материал без разрушения композиции.

**Что фиксировали:**
- Модель: SDXL Lightning 8step  
- Workflow: w07_img2img_inpaint_outpaint.json  
- Seed: 24071377  
- Denoise тесты: 0.35 / 0.55 / 0.75  
- Выбранный вариант: 0.55

**Что меняли:**
- Denoise = 0.55  
- Positive prompt: industrial sci-fi engineer suit concept art  
- Negative prompt: neon, glossy plastic, cartoon style

**Вывод:**
При 0.55 улучшилась читаемость металла и свет, сохранился силуэт.  
При 0.75 появились артефакты и потеря деталей.  
Оставляем 0.55 как baseline.

## Шаг 2. Inpaint test — замена фона

**Контекст:**
Использована маска `mask3.png` (фон белый, персонаж чёрный).  
Проблема: фон слишком нейтральный, не поддерживает технический стиль.  
Цель: заменить фон на индустриальный интерьер.

**Что фиксировали:**
- Модель: SDXL Lightning 8step  
- Workflow: w07_img2img_inpaint_outpaint.json  
- Denoise: 0.75  
- Positive prompt: replace background with industrial sci-fi environment  
- Negative prompt: neon colors, glossy plastic, cartoon style

**Вывод:**
Фон заменён на индустриальный интерьер с потолочными панелями и бетонными стенами.  
Персонажи остались нетронутыми, освещение согласовано.  
Маска сработала корректно.

## Шаг 3. Outpaint test — расширение сцены

**Контекст:**
Исходный кадр после inpaint.  
Проблема: тесная композиция, нет архитектурного контекста.  
Цель: расширить сцену влево и вверх, добавить глубину.

**Что фиксировали:**
- Модель: SDXL Lightning 8step  
- Workflow: w07_img2img_inpaint_outpaint.json  
- Denoise: 0.86  
- Positive prompt: extend industrial sci-fi environment  
- Negative prompt: neon purple, fantasy scenery, organic shapes

**Вывод:**
Композиция расширена, появились потолочные панели и архитектурные детали.  
Сцена стала более пространственной, сохранив технический стиль.
