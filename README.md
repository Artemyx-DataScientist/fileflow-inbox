# FileFlow

**Local-first, reversible file triage for Windows.**

FileFlow is a planned desktop app for turning Downloads and other folders into reviewable file inboxes. It combines ordinary rules, searchable file metadata, optional content extraction, and optional on-device AI. The file system stays the source of truth; decisions and file operations should remain visible and recoverable.

> **Project status: concept / pre-MVP.** This repository currently contains product direction and design constraints. There is no runnable application or release yet.

## Product direction

FileFlow is intended to help people decide what a file is and where it belongs, rather than sorting everything by extension.

- **Rules first, AI when useful.** Handle obvious cases with deterministic rules. Use semantic classification only when the file's meaning matters.
- **Keep uncertain cases reviewable.** High-confidence actions may be automated according to user settings; ambiguous cases go to an inbox for approval.
- **Preview and recover.** Show planned changes before applying them and keep an operation journal so moves can be reviewed and reversed safely.
- **Organize files virtually too.** A file should be discoverable by several categories or tags without requiring a matching physical folder tree.
- **Local-first.** Core file handling should work without a cloud account. An optional local model provider such as Ollama may improve semantic classification.

## Planned processing flow

1. Watch user-selected folders, beginning with Downloads.
2. Wait until a new or changed file is stable before processing it.
3. Record metadata and, where enabled, extract searchable text through format-specific handlers.
4. Apply deterministic rules, then optionally ask a configured local model to classify cases that need semantic context.
5. Automatically handle only cases allowed by the user's rules and confidence thresholds; send the rest to a review queue.
6. Preview and journal every file operation, with a path to undo supported moves.

## MVP goals

- [ ] Watch one configured inbox and detect completed downloads.
- [ ] Provide an incoming queue and a dry-run preview.
- [ ] Support explicit, deterministic rules and manual destination selection.
- [ ] Require confirmation for ambiguous classifications.
- [ ] Record file moves in a journal and support safe undo.
- [ ] Never delete or execute files as part of sorting.

Content extraction, OCR, optional Ollama classification, multi-folder workflows, and virtual categories are later milestones—not current features.

## Provisional architecture

The initial direction is **C# / .NET with WinUI 3**, **SQLite with FTS5** for the local index, optional **Ollama** integration, and pluggable content handlers. These choices are provisional and will be validated before implementation.

## Future OCR and document understanding

High-quality OCR is a research direction beyond the MVP. The goal is accurate recognition of Russian and other documents while preserving their structure, rather than merely attaching one OCR engine. Quality comparable to ABBYY FineReader is a benchmark to investigate, not a claim about FileFlow today.

The proposed subsystem has three parts:

1. **Document analysis.** Prepare scans when needed, detect text regions, tables, columns, and images, and retain reading order and coordinates.
2. **Replaceable OCR engines.** Evaluate local models such as [PaddleOCR-VL](https://github.com/PaddlePaddle/PaddleOCR) and [GLM-OCR](https://github.com/zai-org/GLM-OCR); select or combine engines according to the document and allow targeted re-recognition.
3. **Review and reconstruction.** Let users compare recognized text with the source, inspect uncertain or conflicting results, and correct them. Explore searchable PDF copies and structured output while preserving the original file.

Missing confidence scores must be treated as unknown, not as evidence of a correct result. Dates, document numbers, names, amounts, and other critical fields must not be silently invented or changed by a language model.

Before selecting an engine, compare candidates on representative Russian-language scans and mixed-quality documents, including forms, tables, stamps, and complex layouts. Measure character and word error rates, exact recognition of critical fields, layout and reading order, and practical resource use. Where a properly licensed copy is available, compare results with ABBYY FineReader on the same material.

[PdfCraft](https://github.com/storytold/pdfcraft) and [Folio-OCR](https://github.com/vorojar/Folio-OCR) are references for preprocessing, review, and document workflows. PdfCraft itself describes OCR beyond Latin scripts as a weak area, so it is not assumed to solve Russian recognition. Engine, model-weight, and other asset licenses will be assessed separately before any reuse.

## Safety and privacy principles

- File contents are untrusted data, never instructions for an AI model.
- File operations stay within folders explicitly configured by the user.
- The organizer must not silently delete, overwrite, or execute files.
- AI suggestions are advisory; user-defined rules and approval settings control actions.
- No cloud service is required for the core workflow.

## Inspiration and licensing

FileFlow draws design ideas from these projects. They are references, not dependencies; the ideas below guide the design and are not implemented features yet.

| Project | Ideas informing FileFlow |
| --- | --- |
| [Mouzi](https://github.com/hsr88/mouzi) | A lightweight tray workflow, folder-level modes, local activity history, and undo. |
| [FolderFresh](https://github.com/Trihedron1240/FolderFresh) | Expressive AND/OR rules, profiles, watched folders, previews, and recovery-oriented controls. |
| [organize](https://github.com/tfeldmann/organize) | Composable rules and filters, metadata/content handlers, dry-run simulation, and extension points. |
| [Desktop Janitor](https://github.com/electleaf/Desktop-Janitor) | Stable-file detection, local Ollama classification constrained to known categories, confidence-based review, and an idempotent ledger with undo. Its reviewed repository revision declares no license, so it is an architecture reference only. |
| [AI File Butler](https://github.com/ForgeLabsSoft/ai-file-butler) | Local document extraction and OCR, previews before changes, and using user corrections to improve later suggestions. |
| [DropIt](https://www.dropitproject.com/) | Flexible filter-to-action rules, profiles, and watched-folder workflows as a reference for future extensibility. |
| [PdfCraft](https://github.com/storytold/pdfcraft) | Scan preprocessing, OCR review, and searchable PDF workflows as references for document processing. |
| [Folio-OCR](https://github.com/vorojar/Folio-OCR) | Layout-aware batch OCR and document review as references for local workflows. |

FileFlow combines these directions into a rule-first flow, optional local semantic classification, a review queue for uncertain cases, and visible, journaled operations that can be reversed where supported.

These projects are design references. This repository contains no code or assets copied from them. Any future dependency or code reuse will be reviewed for license compatibility, and required notices will be preserved.

The project is licensed under the [MIT License](LICENSE).

## Contributing

The project is at the requirements and architecture stage. For now, GitHub Issues are the best place to discuss use cases, safety requirements, and design proposals. Build and contribution instructions will be added when the application scaffold exists.

---

# Русский

**Локальный диспетчер файлов для Windows с предварительным просмотром и возможностью отмены действий.**

FileFlow — проектируемое настольное приложение, которое превращает папку «Загрузки» и другие выбранные каталоги во входящие очереди файлов. Оно должно объединить обычные правила, поиск по метаданным, извлечение содержимого и необязательную локальную ИИ-классификацию. Файловая система остаётся источником истины, а решения и операции с файлами должны быть видимыми и обратимыми.

> **Состояние проекта: концепция, до MVP.** Сейчас репозиторий фиксирует направление продукта и требования к его устройству. Готовой программы и релиза пока нет.

## Направление продукта

FileFlow должен помогать определять назначение файла и место для него, а не просто раскладывать всё по расширениям.

- **Сначала правила, ИИ — по необходимости.** Очевидные случаи обрабатываются детерминированными правилами. Семантическая классификация нужна, когда важно содержание файла.
- **Неоднозначные случаи остаются на проверку.** Автоматически можно обрабатывать только случаи, разрешённые настройками пользователя; сомнительные попадают во входящую очередь.
- **Предварительный просмотр и восстановление.** Перед выполнением показываются запланированные изменения, а журнал операций позволяет проверить и безопасно отменить поддерживаемые перемещения.
- **Виртуальная организация поверх папок.** Один файл можно находить по нескольким категориям и тегам, не копируя его и не создавая для каждого признака отдельную физическую папку.
- **Локальная работа.** Для основного процесса не нужна облачная учётная запись. Необязательный локальный провайдер, например Ollama, может улучшить семантическую классификацию.

## Планируемый конвейер

1. Наблюдать за выбранными пользователем папками, начиная с «Загрузок».
2. Дождаться завершения загрузки или изменения файла.
3. Сохранить метаданные и, если это включено, извлечь текст через обработчики форматов.
4. Применить обычные правила и при необходимости передать неоднозначный случай настроенной локальной модели.
5. Автоматически обрабатывать только случаи, разрешённые правилами и порогами уверенности; остальные отправлять в очередь проверки.
6. Показывать и журналировать операции с файлами, предусматривая отмену поддерживаемых перемещений.

## Цели MVP

- [ ] Наблюдение за одной входящей папкой и обнаружение завершённых загрузок.
- [ ] Очередь файлов и предварительный просмотр без выполнения действий.
- [ ] Явные детерминированные правила и ручной выбор папки назначения.
- [ ] Подтверждение неоднозначных классификаций.
- [ ] Журнал перемещений и безопасная отмена операций.
- [ ] Никакого удаления или запуска файлов при сортировке.

Извлечение содержимого, OCR, интеграция с Ollama, несколько входящих папок и виртуальные категории — будущие этапы, а не реализованные функции.

## Предварительное направление архитектуры

Начальный вариант — **C# / .NET и WinUI 3**, **SQLite с FTS5** для локального индекса, необязательная интеграция с **Ollama** и подключаемые обработчики содержимого. Эти решения предварительные и будут проверены до начала реализации.

## Будущее OCR и работа со структурой документов

Качественное распознавание — направление исследований после MVP. Цель — точно распознавать русскоязычные и другие документы с сохранением их структуры. Сопоставимость с ABBYY FineReader — ориентир для сравнительных испытаний, а не заявление о достигнутом качестве FileFlow.

Предварительно подсистема делится на три части:

1. **Анализ документа.** При необходимости подготовить скан, определить области текста, таблицы, колонки и изображения, сохранить порядок чтения и координаты фрагментов.
2. **Заменяемые OCR-движки.** Исследовать локальные модели, например [PaddleOCR-VL](https://github.com/PaddlePaddle/PaddleOCR) и [GLM-OCR](https://github.com/zai-org/GLM-OCR); подбирать или сочетать движки с учётом документа и повторно распознавать отдельные области.
3. **Проверка и восстановление.** Дать пользователю сравнить распознанное с исходником, увидеть сомнительные и противоречивые результаты и исправить ошибки. Исследовать создание копий PDF с текстовым слоем и структурированного результата, сохраняя исходный файл.

Отсутствие оценки уверенности означает неизвестный результат, а не подтверждённую правильность. Языковая модель не должна незаметно придумывать или изменять даты, номера документов, фамилии, суммы и другие важные данные.

Перед выбором движка сравним кандидатов на представительных русскоязычных сканах и документах разного качества: с бланками, таблицами, печатями и сложной вёрсткой. Будем оценивать ошибки на уровне символов и слов, точность важных реквизитов, сохранение структуры и порядка чтения, а также требования к ресурсам. При наличии лицензии проведём сравнение с ABBYY FineReader на тех же материалах.

[PdfCraft](https://github.com/storytold/pdfcraft) и [Folio-OCR](https://github.com/vorojar/Folio-OCR) служат ориентирами для предобработки, проверки распознавания и работы с документами. Авторы PdfCraft сами указывают на слабую поддержку OCR вне латиницы, поэтому его качество для русского не считается решённой задачей. Лицензии движков, весов моделей и других материалов будут проверяться отдельно до повторного использования.

## Принципы безопасности и приватности

- Содержимое файлов считается недоверенными данными, а не инструкциями для ИИ-модели.
- Операции выполняются только в папках, явно настроенных пользователем.
- Приложение не должно незаметно удалять, перезаписывать или запускать файлы.
- ИИ предлагает варианты; действия определяются правилами и настройками подтверждения.
- Для основного сценария не требуется облачный сервис.

## Проекты, которые вдохновляют FileFlow

В проекте используются идеи из следующих файловых органайзеров. Это ориентиры для проектирования, а не зависимости; перечисленные возможности пока не реализованы.

| Проект | Что берём как идею для FileFlow |
| --- | --- |
| [Mouzi](https://github.com/hsr88/mouzi) | Лёгкий сценарий через системный трей, режимы для отдельных папок, локальную историю действий и отмену. |
| [FolderFresh](https://github.com/Trihedron1240/FolderFresh) | Выразительные правила с условиями AND/OR, профили, наблюдение за папками, предварительный просмотр и средства восстановления. |
| [organize](https://github.com/tfeldmann/organize) | Составные правила и фильтры, обработчики метаданных и содержимого, пробный запуск без изменений и точки расширения. |
| [Desktop Janitor](https://github.com/electleaf/Desktop-Janitor) | Проверку завершения записи файла, локальную классификацию через Ollama по заданным категориям, порог уверенности, очередь разбора и журнал с отменой. В проверенной версии репозитория лицензия не указана, поэтому это только архитектурный ориентир. |
| [AI File Butler](https://github.com/ForgeLabsSoft/ai-file-butler) | Локальное извлечение текста и OCR, предварительный просмотр действий и использование исправлений пользователя для улучшения будущих подсказок. |
| [DropIt](https://www.dropitproject.com/) | Гибкие правила «фильтр → действие», профили и наблюдение за папками как идеи для расширяемости. |
| [PdfCraft](https://github.com/storytold/pdfcraft) | Предобработка сканов, проверка распознавания и создание PDF с текстовым слоем как ориентиры для обработки документов. |
| [Folio-OCR](https://github.com/vorojar/Folio-OCR) | Пакетное распознавание с учётом структуры страницы и проверка документов как ориентиры для локальной работы. |

FileFlow объединяет эти направления в один процесс: сначала детерминированные правила, затем при необходимости локальная семантическая классификация, очередь для неоднозначных случаев и видимый журнал операций с возможностью отмены там, где она поддерживается.

Эти проекты служат источниками идей. Код и графические материалы из них в репозиторий не копируются. Будущие зависимости или повторное использование кода будут отдельно проверяться на совместимость лицензий, а необходимые уведомления будут сохранены.

Проект распространяется по [лицензии MIT](LICENSE).

## Участие в проекте

Пока проект находится на этапе требований и архитектуры. Для обсуждения сценариев использования, требований безопасности и предложений по устройству проекта используйте GitHub Issues. Инструкции по сборке и внесению изменений появятся вместе с каркасом приложения.
