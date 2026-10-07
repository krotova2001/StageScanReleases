# StageScanDesktop — Desktop Client (WPF)

Приложение для учёта, маркировки и управления оборудованием прокатной компании по звуку.

## Что такое StageScanDesktop

StageScanDesktop — настольное приложение для Windows. Оно помогает звукорежиссёрам, техникам и сотрудникам прокатных компаний быстро работать с парком оборудования:

- находить устройства по QR-коду
- просматривать карточки оборудования
- добавлять и смотреть фотографии
- печатать QR-метки
- вести каталог через встроенную админку

Клиент подключается к облачному API и закрывает повседневные задачи инвентаризации на складе и на площадке.

## Основные возможности

- Сканирование QR-кодов через обычные HID-сканеры (USB / Bluetooth)
- Печать QR-меток на принтерах Brother, SUPVAN и других
- Просмотр и редактирование оборудования
- Добавление фотографий
- Минимальный офлайн-режим (локальный кэш)
- Встроенная админка для управления каталогом

## Платформа

StageScanDesktop — Windows-клиент на **WPF** и .NET 10. Работает на Windows 10/11 и использует встроенную тему Fluent.

## API

Приложение работает с удалённым облачным API: оборудование, фотографии, категории.

**API не входит в этот репозиторий.** Здесь только desktop-клиент.

Базовый адрес задаётся в `src/StageScan.Wpf/appsettings.json`:

```json
{
  "Api": {
    "BaseUrl": "https://localhost:5001/",
    "TimeoutSeconds": 30,
    "ApiKey": ""
  }
}
```

Черновик контракта, который ожидает клиент, — в [docs/api.md](docs/api.md).

## Структура репозитория

```
StageScanDesktop/
├── src/
│   ├── StageScan.Wpf/             # окно, экраны, диалоги
│   ├── StageScan.App/             # ViewModels и абстракции UI
│   ├── StageScan.Core/            # модели, DTO, интерфейсы
│   ├── StageScan.Services/        # API-клиент, кэш, фото, синхронизация
│   └── StageScan.Services.Windows/ # печать Windows
├── tests/
├── docs/
├── tools/
└── StageScanDesktop.sln
```

- `StageScan.Wpf` → `StageScan.App`, `StageScan.Services`, `StageScan.Services.Windows`, `StageScan.Core`
- `StageScan.App` → `StageScan.Core`
- `StageScan.Services` и `StageScan.Services.Windows` → `StageScan.Core`

## Сборка и запуск

Требования: Windows 10 1809+ / Windows 11, .NET 10 SDK.

```powershell
dotnet build StageScanDesktop.sln -c Debug
dotnet run --project src\StageScan.Wpf\StageScan.Wpf.csproj
```

Либо откройте `StageScanDesktop.sln` в Visual Studio и запустите проект `StageScan.Wpf`.

Готовый скрипт: `tools/build-windows.ps1`.

## Сборка и релизы (GitHub Actions)

В репозитории настроены два workflow:

- **CI** (`.github/workflows/ci.yml`) — сборка и тесты при push и pull request в `main`.
- **Release** (`.github/workflows/release.yml`) — публикация Windows-установщика Velopack в [StageScanReleases](https://github.com/krotova2001/StageScanReleases).

### Скачать готовую сборку

1. Откройте страницу [Releases](https://github.com/krotova2001/StageScanReleases/releases).
2. Выберите нужную версию.
3. Скачайте `Setup.exe` и запустите его.

Установщик ставит приложение в профиль пользователя (права администратора не нужны) и добавляет ярлык в меню «Пуск». Отдельная установка .NET не требуется. Кэш, фото и настройки остаются в `%LocalAppData%\StageScanDesktop` и не затираются при обновлении программы.

### Создать новый релиз

**Через git-тег** (рекомендуется):

```powershell
git tag v0.1.0
git push origin v0.1.0
```

После push тега workflow соберёт приложение и опубликует релиз автоматически.

**Вручную через GitHub:**

1. Вкладка **Actions** → workflow **Release** → **Run workflow**.
2. Укажите версию (например `0.1.0`) и запустите.

### Локальная публикация

```powershell
./tools/publish-windows.ps1 -Version 0.1.0
./tools/pack-velopack.ps1 -Version 0.1.0
```

Результат: папка `artifacts/publish/Release/win-x64/` и установщик Velopack в `artifacts/velopack/`.
