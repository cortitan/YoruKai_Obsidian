# BoxTutorialPrompt — «Коробочная» система подсказок

> Цель: простая и повторяемая схема показа туториальных подсказок на экране, с очередью, приоритетами и автозакрытием.

---

## 1) Что уже работает

**Активы и данные**

- **DT_TutorialPrompts** (DataTable / `STR Tutorial Prompt`):
    
    - `ID` (Name)
        
    - `Text` (Text)
        
    - `DisplayTime` (Float)
        
    - `AutoFadeOut` (Bool)
        
    - `CompletionTag` (Name/Tag)
        
    - `Repeatable` (Bool)
        
    - `Priority` (Int)
        
    - Примеры строк: `Greeting`, `Move`, `PickSword`, `TUT_TestAuto`.
        

**Виджет** — `WBP_TutorialPrompt`

- Переменные: `PromptText` (Text, бинд в `T_Prompt`), `DisplayTime` (Float), `AutoFadeOut` (Bool), `T_Prompt` (TextBlock), `PromptRoot` (панель), анимации `FadeIn`/`FadeOut`.
    
- Функции:
    
    - **StartDisplay(Text, Duration, Auto)**: пишет в переменные, печатает `[W2a] PromptText now=… | Auto=…`.
        
    - **Show()**: делает `Visible`, ставит Opacity=0 → `PlayAnimation(FadeIn)`, печатает `[W3] Show → Visible + Opacity=1`.
        
    - **ForceFadeOut()**: `StopAllAnimations` → `PlayAnimation(FadeOut, with Finished)` → `SetVisibility(Collapsed)` → печать `[W5] FadeOut finished → Collapsed` → **Call `On Prompt Finished`** (Dispatcher).
        
- Диспетчер: **On Prompt Finished** (сигнатура без параметров).
    

**Менеджер** — `BP_TutorialManager`

- Переменные: `PendingQueue` (Name[]), `CompletedIDs` (Name[]), `CurrentID` (Name), `TransitionInProgress` (Bool), `WidgetRef` (WBP_TutorialPrompt ref), `CinematicLock` (Bool).
    
- BeginPlay: `CreateWidget(WBP_TutorialPrompt)` → `WidgetRef = ReturnValue` → `AddToViewport(Z=1000)` → `SetVisibility(Collapsed)` → **подписка на событие виджета** → печать `[M0] Created Widget | OwningPlayer=Valid`.
    
- **ShowTutorial(ID)**: кладёт ID в очередь (если можно), печатает `[M1] ShowTutorial ID={ID}`.
    
- **TryShowNext()**: печатает `[M2] TryShowNext | TransitionInProgress={S} | QueueLen={Q}`; выбирает элемент по приоритетам; печатает `[M3] Dequeue ID={ID}`; затем вызывает **StartID/StartDisplay**.
    
- **Перед показом** печатает `[M5] Call StartDisplay | Text=… | Duration=… | Auto=…` и поднимает флаги/текущие значения.
    
- **HandlePromptFinished** (ОЖИДАЕМО): если `Repeatable=false` → `AddUnique(CompletedIDs, CurrentID)` → `Set CurrentID=None` → `Set TransitionInProgress=false` → печать `[M6] Widget finished | CurrentID={ID} → Next` → `TryShowNext()`.
    

**Наблюдение в игре**

- Подсказка появляется корректно (FadeIn), `[W3]` печатается.
    
- Автозакрытие работает: после `DisplayTime` печатается `[W5] FadeOut finished → Collapsed` и виджет скрывается.
    

---

## 2) Текущее узкое место

**Симптом:** после скрытия первой подсказки **не запускается** логика завершения в менеджере (не печатается `[M6]`, не идёт показ следующей).

**Причина:** событие **`HandlePromptFinished` не вызывается** — делегат `On Prompt Finished` у виджета **не привязан** (или привязка теряется).

Доказательство: принт в начале `HandlePromptFinished` не появляется, при этом `[W5]` из виджета есть.

---

## 3) Исправление проблемы (минимальные шаги)

### A. Диагностика в виджете (обязательно на время отладки)

В `WBP_TutorialPrompt → ForceFadeOut` **перед** `Call On Prompt Finished`:

1. `Get Display Name (self)` → `FormatText` → `PrintString`: **`[W5-Call] Fire from {Self}`**.
    
2. От синего пина диспетчера `On Prompt Finished` вызови **`IsBound`** → `PrintString`: **`[W5-Call] IsBound={B}`**.
    
3. Вставь **`Delay(0.0)`** между `[W5]` и вызовом `Call On Prompt Finished` (устраняет редкие гонки после смены видимости/анимации).
    

Ожидание: `IsBound=true`. Если `false` — идём к шагу B/C ниже.

### B. Надёжная подписка при старте (рекомендуемый способ)

В `BP_TutorialManager → BeginPlay` используй **`Assign On Prompt Finished`** на `WidgetRef` (а не просто `Bind Event to …`).

- Это создаст **встроенный красный Custom Event** справа на узле Assign — его логика и есть `HandlePromptFinished`.
    
- Добавь принт: **`[BIND] Bound to {DisplayName(WidgetRef)}`**.
    

### C. Подстраховка перед каждым показом

Перед вызовом `WidgetRef.StartDisplay(...)` (в той же цепочке, где `[M5]`):

1. **`Unbind All from On Prompt Finished`** (или `Clear`) на `WidgetRef`.
    
2. Сразу **`Assign On Prompt Finished`** к тому же обработчику (`HandlePromptFinished`).
    

Эта пара защищает от случая, если виджет пересоздали/переиспользовали и старая подписка потерялась.

### D. Порядок в `HandlePromptFinished`

Используй `Sequence`:

- **Then0:**
    
    - (опционально) возьми локальную копию `CurrentID` → `FormatText` → печать **`[M6] Widget finished | CurrentID={ID} → Next`** (печатаем, пока `CurrentID` ещё НЕ очищен).
        
    - `Set CurrentID=None` → `Set TransitionInProgress=false`.
        
- **Then1:** `TryShowNext()`.
    

**Проверка:** теперь после `[W5]` должен появиться `[M6]`, затем снова `[M2]` и пойдёт следующая подсказка.

---

## 4) Карта логов (что и где печатается)

- **[M0]** BeginPlay менеджера: виджет создан и скрыт.
    
- **[M1]** ShowTutorial(ID): запрос на показ.
    
- **[M2]** TryShowNext: статус блокировки и длина очереди.
    
- **[M3]** Dequeue ID=…: изъятие из очереди.
    
- **[M4]** RowFound | Text=… | Time=… | Auto=… (или **[M4X]** Row NOT found …).
    
- **[M5]** Call StartDisplay | Text=… | Duration=… | Auto=… (перед вызовом в виджет).
    
- **[W2a]** PromptText now=… | Auto=… (виджет принял данные).
    
- **[W3]** Show → Visible + Opacity=1 (начало показа).
    
- **[W5]** FadeOut finished → Collapsed (конец показа, вызов делегата).
    
- **[M6]** Widget finished | CurrentID=… → Next (менеджер получил сигнал и пошёл дальше).
    
- Временные отладочные: **[W5-Call] Fire from … / IsBound=…**, **[BIND] Bound to …**.
    

---

## 5) Как воспроизвести и «принять работу»

1. Запустить уровень. Ожидаемые первые логи: `[M0]`, `[M1]` (если первый ID выставлен вручную).
    
2. На первом показе: `[M4]` → `[W2a]` → `[M5]` → `[W3]`.
    
3. По окончании таймера: `[W5]` → `[W5-Call] IsBound=true` → **`[M6]`** → `[M2]` следующего шага.
    
4. Проверить, что **вторая** подсказка появляется.
    

Если пункта 3 нет — включить шаги A–C из раздела «Исправление проблемы».

---

## 6) Частые ошибки

- Подписка сделана через `Bind Event…`, но виджет был пересоздан → делегат потерялся. Решение: **Assign + Clear/Assign перед показом**.
    
- Печать `[M6]` формируется **после** обнуления `CurrentID` → в логе пусто. Решение: формируй строку до `Set CurrentID=None` (или сохрани локально).
    
- В `StartDisplay` перепутан вход `Auto` — в логе `[W2a] Auto=false`, а автоожидалось. Решение: подать красный провод от `Auto Fade Out` из строки таблицы.
    

---

## 7) Что осталось сделать (если понадобится)

- Вернуть «чистый» `ApplyData` (опционально) — сейчас всё идёт через `StartDisplay` и это устраивает.
    
- Добавить обработку ручного закрытия (по кнопке) — вызывать `ForceFadeOut()` и путь тот же.
    
- Защитить `TryShowNext` от гонок: дополнительная проверка `TransitionInProgress`.
    

---

### TL;DR

Всё отображается и скрывается; **не срабатывал переход к следующей подсказке из‑за неподписанного делегата**. Лечится: **Assign On Prompt Finished + Clear/Assign перед показом**, и для надёжности — `IsBound`‑принт и `Delay(0.0)` перед вызовом диспатчера в виджете.