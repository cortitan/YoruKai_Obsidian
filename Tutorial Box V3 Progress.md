# 📘 Руководство: Система подсказок и пропуска кат-сцен в UE 5.6 Обновлено

---

## Глава 1. Общая структура

- **WBP_TutorialPrompt** — универсальный виджет, выполняющий две роли:
    
    - Режим `Tutorial` — отображает обучающие подсказки.
        
    - Режим `SkipHint` — показывает сообщение «Нажмите Enter, чтобы пропустить сцену».
        
- **BP_BoxManager** — управляет подсказками:
    
    - Хранит `CurrentWidget` (ссылка на активный Tutorial-подсказ).
        
    - Показывает подсказки через функцию `TryShowNext`.
        
- **Level Blueprint** — управляет кат-сценами и обработкой клавиши Enter.
    

---

## Глава 2. Работа виджета `WBP_TutorialPrompt`

### Переменные:

- `Mode (ETutorialMode)` — определяет режим (`Tutorial` или `SkipHint`).
    
- `PromptText` — текст подсказки.
    
- `CloseHintText` — приписка «Нажмите Enter, чтобы закрыть».
    
- `SkipHintText` — текст «Нажмите Enter, чтобы пропустить сцену».
    

### Логика:

- В `Event Construct` → `Switch on ETutorialMode`.
    
    - Если `Tutorial` → `TutorialGroup = Visible`, `SkipHintGroup = Collapsed`.
        
    - Если `SkipHint` → наоборот.
        
- Анимации:
    
    - `FadeIn` — плавное появление.
        
    - `FadeOut` — плавное исчезновение.
        

### События:

- `ClosePrompt` — запускает FadeOut и удаляет виджет.
    
- `StartFadeOut` — используется для скрытия SkipHint в конце кат-сцены.
    

---

## Глава 3. Логика в `BP_BoxManager`

### Переменные:

- `CurrentWidget` (WBP_TutorialPrompt) — текущая подсказка.
    
- `CinematicLock (bool)` — true во время кат-сцены.
    

### Функции:

- `TryShowNext` — создаёт новый `WBP_TutorialPrompt (Mode = Tutorial)` и сохраняет его в `CurrentWidget`.
    

---

## Глава 4. Логика в Level Blueprint

### Переменные:

- `URLBoxManager` — ссылка на BP_BoxManager.
    
- `TutorialPrompt` — ссылка на текущий виджет (SkipHint или Tutorial).
    

### На BeginPlay:

- `Get All Actors Of Class (BP_BoxManager)` → `Get(0)` → `Set URLBoxManager`.
    

### Обработчик Enter:

1. `Branch (URLBoxManager → CinematicLock)`
    
    - **False (нет кат-сцены)**:
        
        - `IsValid(TutorialPrompt)` → `ClosePrompt` (Tutorial подсказка).
            
    - **True (идёт кат-сцена)**:
        
        - `IsValid(TutorialPrompt)` → `ClosePrompt` (если что-то висит).
            
        - `Create Widget (WBP_TutorialPrompt, Mode=SkipHint)`
            
        - `Set TutorialPrompt`
            
        - `Add to Viewport`
            
        - `ClosePrompt (TutorialPrompt)` → `Delay(0.5)` → `SkipCinematic`
            
        - `Enable Input`
            

### OnFinished_Event (конец кат-сцены):

- `IsValid(TutorialPrompt)` → `ClosePrompt (TutorialPrompt)`
    
- `BP_BoxManager → TryShowNext`
    
- `Enable Input`
    

---

## Глава 5. Проверка сценариев

### Сценарий 1: Ждём кат-сцену

- Кат-сцена стартует → появляется SkipHint.
    
- Кат-сцена заканчивается → SkipHint исчезает → появляется Tutorial.
    
- Enter работает только для Tutorial.
    

### Сценарий 2: Пропускаем кат-сцену Enter

- Кат-сцена стартует → SkipHint на экране.
    
- Жмём Enter → SkipHint исчезает → кат-сцена прерывается → появляется Tutorial.
    
- Enter работает для подсказок.
    

---

## Глава 6. Проблемы и доработки

### Проблемы:

1. SkipHint иногда остаётся после кат-сцены и исчезает только по Enter.
    
2. Tutorial не появляется автоматически после окончания кат-сцены.
    
3. После пропуска кат-сцены Enter иногда блокируется.
    

### Решения:

1. В `OnFinished_Event` закрывать SkipHint (`ClosePrompt`).
    
2. После `ClosePrompt` в `OnFinished_Event` сразу вызывать `TryShowNext`.
    
3. После `SkipCinematic` и в конце кат-сцены всегда вызывать `Enable Input`.
    

---

⚜️ Таким образом, система станет полностью рабочей: SkipHint исчезает вовремя, Tutorial показывается автоматически, а Enter работает корректно и не блокируется.