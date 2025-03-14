# Проблемы с данными в "Медикаменте"

### Отсутствие централизованного хранилища
Данные лежат в Excel и JPG/PDF на общей папке; нет разграничения прав.

Нет аудита действий (кто открыл файл, кто скопировал).

### Отсутствие контроля доступа на уровне данных
Любой, кто имеет доступ к общей папке, может прочитать все файлы пациентов.

Система аутентификации только доменная (AD), но внутри файлов ни RBAC, ни ABAC.

### Отсутствие шифрования при хранении
Файлы с персональными данными и анализами не зашифрованы.

При утере носителя или доступе через общую сеть высок риск утечки.

### Нет систематического подхода к логированию и мониторингу
Не отслеживаются изменения файлов, их пересылка и открытие.

Нет системы Security Information and Event Management (SIEM) или аналогов.

### Нет механизма быстрого удаления / блокировки данных

Если пациент просит удалить свои данные (право на забвение), это придётся делать вручную во множестве Excel/папок.

### Отсутствие механизма тегирования / классификации

Данные не разделены по критериям чувствительности: «особые категории» (мед. тайна), «общие PII», «финансовые данные» и т.д.

## Варианты решения
| Категория данных               | Описание                                                                 | Предлагаемые меры                                                                 |
|--------------------------------|-------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| Личные данные (PII)            | ФИО, дата рождения, телефон, e-mail, адрес                              | Шифрование на уровне БД или файлов, ограничение доступа                           |
| Медицинские карты / анализы    | Диагноз, результаты анализов, хронические заболевания и т. д.            | Шифрование, строгая ролевая модель (RBAC/ABAC), аудит                             |
| Платёжные данные               | Сумма платежа, номер чека, реквизиты операции, транзакции                | Разграничение доступа, шифрование, безопасные журналы аудита                      |
| Доп. данные о пациенте         | Адрес прописки, место учёбы/работы                                       | Минимизация хранения (хранить только при необходимости)                           |
| Внутренние служебные данные    | ID клиента, служебные поля учётных систем                                | Логический доступ только для системных ролей, шифрование, аудит                   |

- **Шифрование**: может быть на уровне диска (BitLocker, LUKS), на уровне базы (TDE), на уровне полей/столбцов (Column Encryption), или с помощью внешних KMS.
- **Обфускация / анонимизация**: для тестовых сред, для BI-аналитики, когда не требуется видеть реальное ФИО.
- **Ограничение ретеншна (Data Minimization)**: хранить ровно столько, сколько нужно для оказания услуги; устаревшие данные — удалять.

## Механизм тегирования данных

### Вводим классификацию (примеры уровней):

- **Public** – общедоступные данные.
- **Internal** – служебная информация.
- **Confidential** – персональные данные.
- **Strictly Confidential** – особые категории (здоровье, паспорт, финансы).

### Используем инструменты тегирования

Это может быть:

- **Microsoft Purview Information Protection** – если используется Microsoft-экосистема, позволяет автоматически классифицировать и ставить метки в файлах Office.
- **Kubernetes/Cloud Labeling** – если данные хранятся в контейнерах, можно проставлять метаданные.
- **Data Catalog** (например, *Collibra*) – ведение сквозных метаданных о том, где лежат файлы, в каких полях какие данные, и установка тегов (*"PII"*, *"Medical"*).

### Автоматизация

Настроить сканеры содержимого (*DLP / MIP / регулярные выражения*), чтобы файлы с персональными данными автоматически получали нужную метку.

### Политики обработки

В зависимости от тега, система должна:

- **Запрещать пересылку наружу** (*для Strictly Confidential*).
- **Требовать шифрования при хранении** (*для Confidential*).
- **Вести аудит действий** (*кто открыл/изменил*).


## Список инструментов, способов и мер, обеспечивающих конфиденциальность

Ниже примерный перечень, который вы можете адаптировать.

### DLP-система (*Data Loss Prevention*)

- Сканирует сетевые ресурсы, почту, файлы.
- Выявляет утечки конфиденциальных данных.

### Система классификации и защиты документов (*MS Purview или аналоги*)

- Автоматическая расстановка меток (*labels*).
- Применение политик защиты.

### Шифрование на уровне хранилищ

- **BitLocker** (*Windows Server*), **LUKS** (*Linux*), аппаратное шифрование на *SAN*.
- **Шифрование файлов и папок** для *Excel/PDF*.

### RBAC или ABAC

- **RBAC (Role-Based Access Control)** – разрешения на просмотр/редактирование файлов только определённым ролям (*Врач, Кассир, Бухгалтер и т. д.*).
- **ABAC (Attribute-Based Access Control)** – проверка контекста (*например, «Врач может видеть данные только своих пациентов», «Доступ возможен только с IP-адреса офиса»*).

### Аудит доступа (*логирование в SIEM*)

- Отслеживание всех событий: *открытие файлов, удаление, копирование*.
- Генерация оповещений при аномальной активности.

### Политика минимизации данных

- Хранить только то, что нужно для оказания услуги.
- Удалять или анонимизировать данные по прошествии сроков хранения.

### Маскирование данных для аналитики

- При выгрузке больших массивов для *BI/ML* скрывать реальные *ФИО*, заменяя псевдонимами или *ID*.
- Чувствительные поля (*паспорт, адрес*) хранить зашифрованно.
- Для аналитики использовать усечённые/хешированные значения.