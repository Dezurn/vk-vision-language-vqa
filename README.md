# Русскоязычный Visual Question Answering на данных VK

## Автор

- ФИО: Максим Литвак
- Учебная группа: ББИ234
- Образовательная программа: Бизнес-информатика, НИУ ВШЭ
- Формат выполнения: индивидуальный проект
- Проект: онлайн-практика VK Education

Проект исследует, дообучает и сравнивает визуально-языковые модели на открытых русскоязычных наборах **GQA-ru** и **MMBench-ru** из коллекции VK/DeepVK. В работе проверены две базовые архитектуры — Qwen2.5-VL-3B-Instruct и SmolVLM-Instruct — и четыре LoRA-адаптера. Итоговая оценка выполнена на всех доступных примерах выбранных оценочных сплитов: 12 216 заданий GQA-ru `testdev` и 3 910 заданий MMBench-ru `dev` для каждой модели.

## Цель и задачи

Цель — повысить качество русскоязычного ответа на вопросы по изображениям с помощью параметрически эффективного дообучения LoRA и определить наиболее сильную конфигурацию на каждом бенчмарке.

В проекте решены следующие задачи:

1. Проведён разведочный анализ GQA-ru, включая структуру данных, длины вопросов и ответов, баланс типов вопросов и покрытие ответов между сплитами.
2. Получены базовые результаты Qwen2.5-VL и SmolVLM.
3. Обучены три LoRA-конфигурации Qwen и одна LoRA-конфигурация SmolVLM.
4. Все шесть вариантов моделей оценены на полных сплитах GQA-ru `testdev` и MMBench-ru `dev`.
5. Сохранены предсказания, агрегированные метрики и разрезы по категориям.

## Как использованы данные VK

- **GQA-ru** используется для обучения LoRA на `train_balanced` и оценки на `testdev_balanced`. Модель получает изображение и вопрос на русском языке и генерирует короткий открытый ответ.
- **MMBench-ru** используется только для оценки переноса качества на задания с вариантами ответа. Из ответа модели извлекается выбранная буква, после чего считается accuracy.

Исходные данные в репозиторий не включены: они скачиваются напрямую из [коллекции DeepVK Vision-Language Modeling](https://huggingface.co/collections/deepvk/vision-language-modeling-664dd7e4c257cc78e740f6bc).

## Итоговые результаты

В таблице приведены метрики полного запуска. Для каждой модели обработано 12 216 заданий GQA-ru и 3 910 заданий MMBench-ru; пропущенных идентификаторов, дубликатов и ошибок инференса в итоговых CSV нет.

| Модель | GQA-ru Exact Match | GQA-ru BERTScore F1 | MMBench-ru Accuracy |
|---|---:|---:|---:|
| Qwen2.5-VL + LoRA #2 | **0,5372** | **0,9065** | 0,8033 |
| Qwen2.5-VL + LoRA #3 | 0,5352 | 0,9055 | **0,8061** |
| Qwen2.5-VL + LoRA #1 | 0,5006 | 0,8981 | 0,7980 |
| Qwen2.5-VL baseline | 0,3198 | 0,8064 | 0,7992 |
| SmolVLM + LoRA | 0,2918 | 0,8511 | 0,3655 |
| SmolVLM baseline | 0,2028 | 0,6790 | 0,3734 |

Главный результат: **Qwen2.5-VL + LoRA #2** лучше всех отвечает на открытые вопросы GQA-ru, повышая Exact Match базовой Qwen на 21,74 п.п. На MMBench-ru лучший результат у **Qwen2.5-VL + LoRA #3** — 0,8061, однако все варианты Qwen находятся близко друг к другу. LoRA заметно улучшает SmolVLM на GQA-ru, но не переносит этот прирост на MMBench-ru.

По типам вопросов GQA-ru Qwen + LoRA #2 особенно уверенно отвечает на вопросы об объектах (`object`: 0,8305) и атрибутах (`attribute`: 0,6287). Отношения между объектами остаются более сложной категорией (`relation`: 0,4304). На MMBench-ru сильными категориями в среднем стали identity reasoning и описание сцены, а наиболее трудными — пространственные отношения и предсказание будущего.

## Опубликованные модели

На Hugging Face опубликованы две лучшие итоговые конфигурации. Веса Qwen LoRA #1 и SmolVLM LoRA не публиковались, поскольку они уступили финальным моделям; их метрики и полные предсказания сохранены в репозитории.

| Модель | Основной результат | Hugging Face |
|---|---|---|
| Qwen2.5-VL + LoRA #2 | Лучший GQA-ru Exact Match: **0,5372** | **[Qwen2.5-VL GQA-ru LoRA v2 →](https://huggingface.co/Dezurg/qwen2.5-v2-3b-gqa-ru-lora)** |
| Qwen2.5-VL + LoRA #3 | Лучший MMBench-ru Accuracy: **0,8061** | **[Qwen2.5-VL GQA-ru LoRA v3 →](https://huggingface.co/Dezurg/qwen2.5-v3-3b-gqa-ru-lora)** |

## Конфигурации обучения

Все запуски используют seed 42, один epoch, batch size 1, gradient accumulation 8, BF16 и gradient checkpointing. Обучались только LoRA-параметры; базовые веса оставались замороженными.

| Адаптер | Обучающих примеров | LoRA `r` / `alpha` | Learning rate | Время сохранённого запуска |
|---|---:|---:|---:|---:|
| Qwen LoRA #1 | 5 000 | 8 / 16 | 2e-5 | 1 ч 05 мин |
| Qwen LoRA #2 | 20 000 | 16 / 32 | 1e-5 | 4 ч 44 мин |
| Qwen LoRA #3 | 30 000 | 32 / 64 | 1e-5 | 8 ч 49 мин |
| SmolVLM LoRA | 10 000 | 16 / 32 | 1e-5 | 2 ч 21 мин |

Подробные параметры и функции подготовки мультимодального диалога сохранены в соответствующих ноутбуках. Интерактивные метрики и логи запусков доступны в [MLflow проекта на DagsHub](https://dagshub.com/Dezurn/vk-vision-language-vqa.mlflow):

- [Qwen LoRA #2 — эксперимент 0](https://dagshub.com/Dezurn/vk-vision-language-vqa.mlflow/#/experiments/0)
- [Qwen LoRA #3 — эксперимент 5](https://dagshub.com/Dezurn/vk-vision-language-vqa.mlflow/#/experiments/5)
- [SmolVLM LoRA — эксперимент 2](https://dagshub.com/Dezurn/vk-vision-language-vqa.mlflow/#/experiments/2)

## Структура проекта

```text
vk-vision-language-vqa/
├── notebooks/
│   ├── EDA.ipynb
│   ├── baseline_qwen.ipynb
│   ├── baseline_smolvlm.ipynb
│   ├── train_lora_qwen_v2.ipynb
│   ├── train_lora_qwen_v3.ipynb
│   ├── train_lora_smolvlm.ipynb
│   ├── evaluate_qwen_lora*.ipynb
│   ├── evaluate_smolvlm_lora.ipynb
│   ├── evaluate_mmbench_ru_models.ipynb
│   ├── evaluate_all_models_full.ipynb
│   ├── evaluate_all_models.ipynb
│   └── merge_lora_model.ipynb
├── reports/evaluation/
│   ├── *_predictions.csv
│   ├── *_metrics.csv
│   └── all_models/
│       ├── gqa_scored_predictions.csv
│       ├── gqa_metrics.csv
│       ├── gqa_category_accuracy.csv
│       ├── mmbench_scored_predictions.csv
│       ├── mmbench_metrics.csv
│       └── mmbench_category_accuracy.csv
├── data/                 # создаётся локально, исключено из Git
├── models/               # базовые и объединённые модели, исключено из Git
├── outputs/              # LoRA-адаптеры, исключено из Git
├── Отчет по проектной задаче.pdf
└── README.md
```

Назначение ключевых ноутбуков:

- `EDA.ipynb` — проверка качества данных и графики с выводом после каждой визуализации.
- `baseline_*.ipynb` — короткая базовая оценка моделей на GQA-ru.
- `train_lora_*.ipynb` — обучение LoRA-адаптеров.
- `evaluate_*_lora.ipynb` и `evaluate_mmbench_ru_models.ipynb` — быстрые выборочные проверки; они пишут результаты в `reports/evaluation/sample_runs` и не затирают полный прогон.
- `evaluate_all_models_full.ipynb` — инференс всех шести моделей на всех данных двух оценочных сплитов, контроль полноты и перезапись основных CSV.
- `evaluate_all_models.ipynb` — пересчёт BERTScore, итоговые таблицы, графики и сравнительные выводы по уже собранным полным предсказаниям.
- `merge_lora_model.ipynb` — объединение Qwen LoRA #2 с базовыми весами.

## Требования

Результаты проекта получены в окружении Python 3.11, PyTorch 2.5.1 + CUDA 12.1 и на GPU NVIDIA GeForce RTX 3080. Для воспроизведения обучения и полного инференса практически необходима CUDA-совместимая GPU; расчёт итоговых таблиц и EDA можно выполнить на CPU.

Рекомендуется иметь не менее 40 ГБ свободного места. Фактический объём зависит от выбранных файлов SmolVLM, кеша Hugging Face и сохранённых чекпоинтов.

## Установка окружения

Команды ниже приведены для PowerShell из корня проекта.

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

Установите PyTorch под свою версию CUDA командой из [официального конфигуратора PyTorch](https://pytorch.org/get-started/locally/). Для окружения, в котором получены результаты проекта, использовалась такая комбинация:

```powershell
python -m pip install torch==2.5.1 torchvision==0.20.1 --index-url https://download.pytorch.org/whl/cu121
```

Установите остальные зависимости:

```powershell
python -m pip install "transformers>=4.49,<6" "accelerate>=1,<2" "peft>=0.14,<1" qwen-vl-utils "datasets>=3,<6" pandas pyarrow numpy pillow tqdm plotly matplotlib bert-score "dagshub>=0.6,<1" "mlflow>=2,<4" huggingface_hub jupyterlab nbformat ipykernel ipywidgets
```

Зарегистрируйте окружение как ядро Jupyter и проверьте CUDA:

```powershell
python -m ipykernel install --user --name vk-vqa --display-name "Python (vk-vqa)"
python -c "import torch; print(torch.__version__, torch.cuda.is_available(), torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

## Скачивание датасетов и базовых моделей

CLI `hf` устанавливается вместе с `huggingface_hub`. Публичные репозитории можно скачать без токена; если Hub запросит авторизацию, выполните `hf auth login`.

```powershell
hf download deepvk/GQA-ru --type dataset --local-dir data/GQA-ru
hf download deepvk/MMBench-ru --type dataset --local-dir data/MMBench-ru

hf download Qwen/Qwen2.5-VL-3B-Instruct --local-dir models/Qwen2.5-VL-3B-Instruct
hf download HuggingFaceTB/SmolVLM-Instruct --local-dir models/SmolVLM-Instruct --exclude "onnx/*" --exclude "*.onnx" --exclude "*.pdf" --exclude "*.png"
```

После загрузки должны существовать пути:

```text
data/GQA-ru/train_balanced_images/train-00000-of-00001.parquet
data/GQA-ru/train_balanced_instructions/train-00000-of-00001.parquet
data/GQA-ru/testdev_balanced_images/testdev-00000-of-00001.parquet
data/GQA-ru/testdev_balanced_instructions/testdev-00000-of-00001.parquet
data/MMBench-ru/mmbench_ru_dev.parquet
models/Qwen2.5-VL-3B-Instruct/
models/SmolVLM-Instruct/
```

LoRA-веса не скачиваются этой командой: их создают обучающие ноутбуки либо их нужно поместить из архива проекта в:

```text
outputs/qwen_lora_adapter/
outputs/qwen_lora_adapter_2/
outputs/qwen_lora_adapter_3/
outputs/smolvlm_lora_adapter/
```

## Порядок воспроизведения

1. Запустите `notebooks/EDA.ipynb`, чтобы проверить структуру локально скачанных данных.
2. При необходимости выполните `baseline_qwen.ipynb` и `baseline_smolvlm.ipynb` для короткой проверки окружения.
3. Обучите адаптеры в `train_lora_qwen_v2.ipynb`, `train_lora_qwen_v3.ipynb` и `train_lora_smolvlm.ipynb`. Для этих ноутбуков настройте DagsHub/MLflow в их конфигурационной ячейке и выполните `dagshub login`.
4. Для полного воспроизведения итоговых CSV запустите `evaluate_all_models_full.ipynb`. Ноутбук обрабатывает все 12 216 + 3 910 заданий для каждой доступной модели, сохраняет промежуточные чекпоинты и может продолжить прерванный запуск.
5. После завершения инференса запустите `evaluate_all_models.ipynb`: он проверит полноту идентификаторов, пересчитает текстовые метрики GQA-ru и построит итоговые сравнения.
6. Для самостоятельной модели с объединёнными весами выполните `merge_lora_model.ipynb`.

Запуск ноутбука из командной строки выглядит так:

```powershell
python -m jupyter nbconvert --to notebook --execute notebooks/EDA.ipynb --output EDA.executed.ipynb --ExecutePreprocessor.timeout=-1
```

Полный инференс занимает много времени и памяти, поэтому его удобнее запускать интерактивно в JupyterLab:

```powershell
python -m jupyter lab
```

## Метрики и проверка корректности

- **GQA-ru Exact Match** — доля ответов, совпавших с эталоном после приведения к нижнему регистру, удаления лишних пробелов и пунктуации.
- **GQA-ru BERTScore F1** — семантическое сходство с эталонным ответом на основе `bert-base-multilingual-cased`.
- **MMBench-ru Accuracy** — доля правильно выбранных вариантов ответа.

Перед построением итоговых таблиц ноутбуки проверяют число уникальных идентификаторов, дубликаты, пропуски, лишние строки и ошибки инференса. Различия менее одного процентного пункта на MMBench-ru следует трактовать осторожно: несколько конфигураций Qwen показывают практически одинаковое качество.

## Артефакты решения

В `reports/evaluation` сохранены полные предсказания каждой модели и её метрики. Папка `reports/evaluation/all_models` содержит объединённые таблицы и категориальные разрезы, на которых основаны числа в README и выводы `evaluate_all_models.ipynb`.

Датасеты, исходные веса, неопубликованные LoRA-адаптеры и временные чекпоинты намеренно исключены из Git из-за размера. Две лучшие объединённые Qwen-модели опубликованы на Hugging Face. Репозиторий содержит воспроизводимый код, метрики и полные предсказания всех оценённых конфигураций.

## Ограничения интерпретации

- GQA-ru измеряет качество коротких открытых ответов, а MMBench-ru — выбор варианта; их численные метрики нельзя напрямую усреднять.
- Exact Match не засчитывает корректный синоним, если он не совпал с эталоном после нормализации, поэтому рядом приведён BERTScore.
- `testdev` GQA-ru содержит много вопросов к одним и тем же 398 изображениям; число строк не равно числу независимых изображений.
- MMBench-ru `dev` является открытым оценочным сплитом, поэтому приведённые значения относятся именно к этой версии данных.

## Ссылки

- [Репозиторий проекта на GitHub](https://github.com/Dezurn/vk-vision-language-vqa)
- [Репозиторий проекта на DagsHub](https://dagshub.com/Dezurn/vk-vision-language-vqa)
- **[Итоговая модель Qwen2.5-VL + LoRA #2 на Hugging Face](https://huggingface.co/Dezurg/qwen2.5-v2-3b-gqa-ru-lora)** — лучшая по GQA-ru Exact Match.
- **[Итоговая модель Qwen2.5-VL + LoRA #3 на Hugging Face](https://huggingface.co/Dezurg/qwen2.5-v3-3b-gqa-ru-lora)** — лучшая по MMBench-ru Accuracy.
- [Логи обучения и эксперименты на DagsHub](https://dagshub.com/Dezurn/vk-vision-language-vqa.mlflow)
- [Коллекция DeepVK Vision-Language Modeling](https://huggingface.co/collections/deepvk/vision-language-modeling-664dd7e4c257cc78e740f6bc)
- [Vision Language Models Explained](https://huggingface.co/blog/vlms)
- [Qwen2.5-VL-3B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-3B-Instruct)
- [SmolVLM-Instruct](https://huggingface.co/HuggingFaceTB/SmolVLM-Instruct)
