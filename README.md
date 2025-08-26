<div align="center">

# 🛡️ UnFairHire

### Нечестные собеседования с обманом

FH должен обеспечивать прозрачность интервью, отслеживая действия кандидата в режиме реального времени и предотвращая списывание, **но не получилось**

---

</div>

## Dislaimer

Все действия в по гайду выполняются на вашей собственной ответственности. Мы не несем ответственности за любые последствия, связанные с использованием этого репозитория. Все содержимое и изменения в этом репозитории сделаны только в учебных целях.

# Как начать патчить приложение?

1. Скачиваем официальный репозиторий FH.

## Как убрать свой процесс из отправки на сервер?

1. Открываем `apps/tracker-electron/src/features/activity-tracking/process-tracking/const.ts`
2. Удаляем свой процесс из списка `suspiciousProcesses` и добавляем его в список `systemProcessesPatterns`

## Как отключить скрытие при создании скриншотов?

1. Открываем `apps/tracker-electron/src/features/screenshot-protection/screenshot-protection.service.ts`
2. Ищем функцию `applyNativeProtection`, ищем там строку `window.setContentProtection(true);` и удаляем её либо заменяем на `true` на `false`
3. Ищем функцию `applyMacOSProtection` и удаляем там `window.setHasShadow(false);`, `window.setOpacity(0.99);` и `window.setHasShadow(true);`
4. Ищем функцию `applyWindowsProtection` и удаляем там ` window.webContents.applyWindowsProtection(false);`

## Как отключить уведомление при создании скриншотов?

1. Открываем `apps/tracker-electron/src/features/activity-tracking/unified-activity-tracker.ts`
2. Ищем функции `registerScreenshotShortcuts` и удаляем все из блока `try {}` внутри неё

## Как отключить детект виртуальных машин?

1. Открываем `apps/tracker-electron/src/features/vm-detection/vm-detection.service.ts`
2. Ищем функцию `detectVMWithSystemInformation`, `detectVMWithSystemCommands`, `detectVMLinux`, `detectVMWindows`, `detectVMMacOS` и смело удаляем их.
3. Ищем функцию `detectVM` и заменяем её на 
```ts
  public async detectVM(): Promise<VMDetectionResult> {
    const vmResult: VMDetectionResult = {
      isVirtual: false,
      detectionMethod: "systeminformation",
    };

    const resourceInfo = await this.getSystemResources();
    vmResult.systemResources = resourceInfo;

    return vmResult;
  }
```

## Как собрать патченное приложение и лить данные на сервер FH?

Почему это работает и как: бэкенд на Supabase, для работы приложения обязательно нужно указать SUPABASE_URL и SUPABASE_ANON_KEY, по стандарту они являются публичными, так что можно спокойно найти их в собранном приложении FH или отснифать при запросах.

Скрыть их не получится, так как uglify и подобные инструменты не поддерживают xor или подобные операции со строками.

### macOS

1. Скачиваем и устанавливаем оригинальный клиент FH.
2. На macOS открываем папку `Applications`, жмем ПКМ на FH и выбираем `Show Package Contents`.
3. Открываем папку `Contents/Resources` и копируем (!!, не переносим ссылку на рабочий стол) файл `app.asar` себе на рабочий стол.
4. Открываем терминал и пишем `cd ~/Desktop`
5. Пишем `npx @electron/asar extract app.asar fh` (так мы распаковываем данные из приложения FH)
6. Открываем созданную папку `fh` и в ней открываем папку `dist`, нам нужен файл `env.js`, открываем его в редакторе и копируем содержимое
7. Открываем `apps/tracker-electron/src/features/activity-tracking/config.ts` и вставляем вместо `SUPABASE_URL` ссылку из `env.js` и `SUPABASE_ANON_KEY` из `env.js`
8. Теперь нужно собрать приложение, для этого открываем терминал и устанавливаем зависимости `bun i` (либо любой другой менеджер пакетов) в корне репозитория.
9. Запускаем сборку приложения при помощи `bun run tracker:release:mac`
10. После сборки приложение будет в папке `dist/apps/tracker-electron/release`
11. Запускаем приложение и проверяем, что все работает.

### Windows

Когда-нибудь напишу, пока нет возможности тестировать на Windows.

### Linux

Когда-нибудь напишу, пока нет возможности тестировать на Linux.