# Удаление фона в реальном времени

## Выбранное решение

В проекте используется **MediaPipe Selfie Segmentation** для сегментации человека и **OpenCV** для захвата видео, композитинга, отображения результата и сохранения demo-видео.

- Документация MediaPipe Selfie Segmentation:
  https://chuoling.github.io/mediapipe/solutions/selfie_segmentation.html
- Python-пакет MediaPipe:
  https://pypi.org/project/mediapipe/

## Описание архитектуры

Выбранная модель относится к классу легковесных **CNN** и построена на базе **MobileNetV3**. В документации MediaPipe описаны две основные конфигурации:

- `general`: вход `256x256`
- `landscape`: вход `144x256`, меньше вычислений и выше скорость

В этой работе используется `model_selection=1`, то есть `landscape`-вариант, потому что он лучше подходит для работы в реальном времени на CPU.

## Почему это решение подходит для задачи

- работает локально на **CPU**;
- не требует обучения своей модели;
- из коробки хорошо интегрируется с Python и OpenCV;
- даёт мягкую маску, которую удобно сглаживать и использовать для композитинга;
- обеспечивает высокий FPS на входном разрешении `640x480`.

## Что находится в папке

- `background_removal.ipynb` - основной ноутбук с реализацией, benchmark и ячейками запуска
- `requirements.txt` - зафиксированные зависимости
- `assets/abstract_studio_background.png` - картинка для режима замены фона
- `demo/background_removal_demo.mp4` - короткое demo-видео

## Установка

```bash
git clone git@github.com:Azamat-Usmanov/deep-cv-hw2.git
cd deep-cv-hw2
python3.10 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Запуск

Откройте `background_removal.ipynb`, выполните ячейки с импортами и определениями, затем используйте одну из ячеек запуска.

### Пример запуска с веб-камеры

```python
run_realtime_demo(
    source=0,
    background_mode="image",
    background_image_path=DEFAULT_BACKGROUND_PATH,
    width=640,
    height=480,
    inference_width=256,
    infer_every_n=1,
    flip_camera=True,
    show_window=True,
)
```

### Пример запуска на видеофайле

```python
run_realtime_demo(
    source="path/to/video.mp4",
    background_mode="blur",
    width=640,
    height=480,
    inference_width=256,
    infer_every_n=1,
    flip_camera=False,
    show_window=True,
)
```

## Поддерживаемые режимы визуализации

- `background_mode="image"` - замена фона на изображение
- `background_mode="color"` - замена фона на однотонный цвет
- `background_mode="blur"` - размытие исходного фона

## Основные параметры

- `source` - `0` для камеры или путь к видеофайлу
- `width`, `height` - разрешение, в котором кадр подаётся в пайплайн
- `background_mode` - `image`, `color` или `blur`
- `background_image_path` - путь к картинке для режима `image`
- `background_color` - BGR-кортеж для режима `color`
- `inference_width` - ширина кадра после уменьшения перед сегментацией
- `infer_every_n` - выполнять сегментацию один раз в `N` кадров
- `flip_camera` - отражать кадр с камеры по горизонтали
- `show_window` - показывать результат в окне OpenCV
- `output_path` - сохранить итоговое видео на диск

## Как измеряется производительность

Замер сделан функцией `benchmark_processed_fps(...)`.

Так же важно, что **FPS считается только по обработанным кадрам**, то есть по времени, потраченному на сегментацию и композитинг (`remover.process(frame)`). Частота захвата камеры, скорость чтения файла и скорость отрисовки окна в этот показатель не включаются.

Параметры benchmark в ноутбуке:

- warmup: `15` кадров
- измеряемые кадры: `180`
- разрешение пайплайна: `640x480`
- режим визуализации: `image`
- ширина кадра для инференса: `256`
- модель: MediaPipe Selfie Segmentation, `model_selection=1`
- устройство: Apple M3 CPU

## Результаты

- Разрешение: `640x480`
- Средний реальный FPS с камеры: `21.73`
- Средняя задержка: `46.03 ms/frame`
- Устройство: `Apple M3`, `macOS 15.3`, `Python 3.10.17`
- FPS источника камеры: `30.0`
- Качество: маска стабильна на лице и корпусе, границы в целом аккуратные, заметны только мягкие ореолы около волос и плеч

## Демо

- Demo-файл в репозитории: [demo/background_removal_demo.mp4](demo/background_removal_demo.mp4)
