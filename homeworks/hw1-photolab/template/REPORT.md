# ДЗ 1 «Фотолаборатория» — отчет

**Автор:** Османов Осман · **Группа / трек:** MFAI-03 · **Дата сдачи:** 11.10.2026

## 1. Числа

Заполняется из консольного вывода команд (JSON-строки). Для каждого числа — файл и параметры.

### 1.1. Гамма (`data/dark.jpg`)

| γ | среднее | медиана | p1 | p99 | обрезано черных, % | обрезано белых, % |
|---|---------|---------|----|-----|--------------------|-------------------|
| исходник | 25.5 | 5 | 0 | 229 | 7.37 | 0.02 |
| 0.5 | 59.0 | 36 | 0 | 242 | 7.3 | 0.02 |
| 2.0 | 10.8 | 0 | 0 | 206 | 61.87 | 0.01 |

### 1.2. Баланс белого (`data/bright.jpg`)

| Метод | усиление B, G, R | средние до | средние после |
|---|---|---|---|
| grey-world | 1.137, 1.008, 0.886 | 98.6, 110.8, 126.0 | 112.1, 111.7, 111.6 |
| white-patch (p99) | 1.25, 1.138, 1.0 | 98.6, 110.8, 126.0 | 123.1, 126.0, 126.0 |

### 1.3. `equalizeHist` против CLAHE (`data/dark.jpg`, clip 2, tiles 8)

| Кадр | σ в тенях | среднее в тенях | среднее в светах |
|------|-----------|-----------------|------------------|
| исходный | 0.4 | 0.8 | 101.0 |
| equalizeHist | 31.9 | 62.9 | 227.4 |
| CLAHE | 1.7 | 4.9 | 135.4 |

### 1.4. Хромакей (`data/object.jpg`, границы H=175...0, S=160...255, V=40...255)

| Что | Значение |
|-----|----------|
| границы lo / hi | 175,160,40 / 0,255,255 |
| доля маски, % | 7.67 |
| компонент до / после морфологии | 112 / 1 |
| дыр | 0 |
| `ball-lamp.jpg` курса: recall / precision | 94.6% / 99.9% |

### 1.5. Геометрия

| Что | Значение |
|-----|----------|
| поворот 30°: размер до → после, пустой холст, % | 2000×1500 → 2482×2299, 47.4% |
| уменьшение ×0.25: PSNR naive / blur / area к Lanczos, свое фото | 33.3 / 37.2 / 40.5 дБ |
| то же на `iu-building.jpg` | 27.9 / 36.0 / 37.2 дБ |
| выпрямление: стороны в кадре (px), `--aspect`, размер выхода | 819.4 / 1046.5 / 1129.4 / 1017.0, 0.707, 1129×1597 |
| `poster-angle.jpg` курса: совпадение с `poster-gt.png`, % | 99.1 |

### 1.6. Контактный лист

N = 20, сетка 4 × 5, размер листа 1320 × 1378 — `out/sheet.jpg`.

## 2. Три вопроса

**2.1. Какое цветовое пространство выбрано для хромакея и почему? Что было бы в RGB?**

Используется HSV, поскольку цветовой тон отделен от яркости. Это позволяет выделять объект при разном освещении. В RGB изменение яркости влияет сразу на три канала, поэтому подобрать границы сложнее.

**2.2. Что происходит с гистограммой при γ < 1 и при γ > 1? Куда деваются обрезанные пиксели?**

При γ < 1 гистограмма смещается вправо, изображение светлеет. При γ > 1 смещается влево, изображение темнеет. При γ = 2 доля черных пикселей выросла с 7.37% до 61.87%. Пиксели не исчезают, малые значения округляются до нуля, а значения яркости ограничены диапазоном 0...255.

**2.3. Почему CLAHE лучше `equalizeHist` на вашем фото — по числам из 1.3? Если не лучше — что измерили и почему?**

CLAHE дает более умеренное осветление. Средняя яркость светов составила 135.4 против 227.4 у `equalizeHist`. Стандартное отклонение в тенях составило 1.7 против 31.9. Глобальная эквализация сильнее увеличивает контраст, но усиливает шум. На данном фото CLAHE выглядит естественнее.

## 3. Где сломалось

Команда, файл, что получилось, причина, что бы помогло.

Автоматический хромакей (`out/object_on_bg_sample.jpg`) плохо выделяет яблоко. Маска покрывает 4.27% кадра и содержит 9 дыр. Причина в том, что красные оттенки находятся по обе стороны границы HSV 0/179. При ручных границах H=175...0, S от 160 и закрытии 65×65 доля маски увеличилась до 7.67%, а число дыр стало равно 0. Результат улучшился, но часть тени и блика все равно теряется.

## 4. Команды

Все, что лежит в `out/`, получено этими командами (скопируйте из консоли):

```bash
python photolab.py gamma data/dark.jpg --gamma 0.5 -o out/dark_g05.jpg --hist out/dark_g05_hist.png

python photolab.py gamma data/dark.jpg --gamma 2 -o out/dark_g20.jpg --hist out/dark_g20_hist.png

python photolab.py wb data/bright.jpg --method grey-world -o out/bright_gw.jpg --hist out/bright_gw_hist.png

python photolab.py wb data/bright.jpg --method white-patch -o out/bright_wp.jpg --hist out/bright_wp_hist.png

python photolab.py clahe data/dark.jpg --clip 2 --tiles 8 -o out/dark_clahe.jpg --compare out/dark_cmp.jpg --hist out/dark_clahe_hist.png

python photolab.py chromakey data/object.jpg --bg data/bg.jpg --sample 1080,840,120,120 -o out/object_on_bg_sample.jpg --mask out/object_mask_sample.png

python photolab.py chromakey data/object.jpg --bg data/bg.jpg --lo 175,160,40 --hi 0,255,255 --open 5 --close 65 --feather 2 -o out/object_on_bg.jpg --mask out/object_mask.png

python photolab.py rotate data/poster.jpg --deg 30 -o out/poster_rot30.jpg

python photolab.py resize data/text.jpg --scale 0.25 --method naive --report -o out/text_x025_naive.jpg

python photolab.py resize data/text.jpg --scale 0.25 --method blur --report -o out/text_x025_blur.jpg

python photolab.py resize data/text.jpg --scale 0.25 --method area --report -o out/text_x025.jpg

python photolab.py rectify data/poster.jpg --corners 1050,315,1835,550,1090,1285,115,715 --aspect 0.707 -o out/poster_front.jpg

python photolab.py sheet data/dark.jpg out/dark_g05.jpg out/dark_g20.jpg out/dark_clahe.jpg data/bright.jpg out/bright_gw.jpg out/bright_wp.jpg data/object.jpg out/object_mask.png out/object_on_bg.jpg out/object_mask_sample.png out/object_on_bg_sample.jpg data/bg.jpg data/poster.jpg out/poster_front.jpg out/poster_rot30.jpg data/text.jpg out/text_x025_naive.jpg out/text_x025_blur.jpg out/text_x025.jpg --cols 4 -o out/sheet.jpg

python photolab.py chromakey ../../../lectures/L03-color-histograms/demo/ball-lamp.jpg --bg ../../../lectures/L04-geometry/demo/road-cam.jpg --lo 140,121,40 --hi 168,230,255 -o out/ball_lamp_course.jpg --mask out/ball_lamp_mask_course.png

python photolab.py resize ../../../lectures/L05-filtering/demo/iu-building.jpg --scale 0.25 --method area --report -o out/iu_building_x025.jpg

python photolab.py rectify ../../../lectures/L04-geometry/demo/poster-angle.jpg --corners 560,330,1250,220,1290,1010,600,840 --size 640x905 -o out/poster_course_front.png
```

## 5. Инструменты

Чем пользовались (ИИ-помощники, чужие фрагменты — со ссылками), что сгенерировано, как проверено.
Раздел не оценивается, но по нему идет разговор при показе.

Использовались Python, OpenCV, NumPy, Pillow, Matplotlib и ChatGPT. Код и отчет делался с помощью gpt.
