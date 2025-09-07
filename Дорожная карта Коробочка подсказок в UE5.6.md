## 🔹 Глава 1. Подготовка системы

### 🎯 Цель

Создать основу: BoxManager (мозг), Widget (лицо), Character (сенсоры), Level BP (режиссёр).

### Шаги

1. В Content Browser создайте папку `TutorialSystem`.
    
2. В ней создайте три основных Blueprint’а:
    
    - **BP_BoxManager (Actor)** – будет жить на уровне.
        
    - **WBP_TutorialPrompt (Widget Blueprint)** – интерфейс подсказки.
        
    - **BP_ThirdPersonCharacter (Character)** – если он уже есть, будем расширять его.
        
3. Подготовьте **DataTable (DT_TutorialPrompts)** на основе структуры `ST_TutorialPrompt`.
    
    - Поля: `ID`, `Text`, `CompletionTag`, `Repeatable`, `Priority`.
        
    - Пример:
        
        - Greeting → Tag = Skip.
            
        - Movement → Tag = Tutorial.Move.
            
        - Jump → Tag = Tutorial.Jump.
            

---

## 🔹 Глава 2. WBP_TutorialPrompt (лицо системы)

### 🎯 Цель

Виджет умеет показывать два режима: Tutorial и SkipHint.

### Шаги

1. В Designer создайте **Canvas**.
    
2. Внутри – два контейнера:
    
    - `TutorialGroup` → текст + приписка «Нажмите Enter, чтобы закрыть».
        
    - `SkipHintGroup` → текст «Нажмите Enter, чтобы пропустить сцену».
        
3. Оба контейнера сверните через `Visibility`.
    

### Логика (Event Graph)

1. **Event Construct → Switch on ETutorialMode**
    
    - Если Tutorial → TutorialGroup = Visible, SkipHintGroup = Collapsed.
        
    - Если SkipHint → наоборот.
        
2. **ClosePrompt (Custom Event)**
    
    - Play Animation (FadeOut).
        
    - Delay(0.5).
        
    - **Call OnPromptClosed** (Event Dispatcher).
        
    - Remove from Parent.
        
3. **Animations**
    
    - FadeIn – плавное появление.
        
    - FadeOut – плавное исчезновение.
        

📌 Подсказка для новичка:  
Если запутаетесь, помните: **Widget – это картинка и кнопка "Я закрыт"**. Он ничего не решает, он только сообщает Manager’у, что он исчез.

---

## 🔹 Глава 3. BP_BoxManager (мозг системы)

### 🎯 Цель

Управляет очередью подсказок.

### Переменные

- `TutorialDataTable (DataTable)`
    
- `PendingQueue (Array of Name)`
    
- `CurrentPromptID (Name)`
    
- `CompletedIDs (Array of Name)`
    
- `CurrentWidget (WBP_TutorialPrompt)`
    
- `CinematicLock (Bool)`
    

### Функции

1. **ShowTutorial(ID)**
    
    - Добавляет ID в очередь.
        
    - Вызывает TryShowNext.
        
2. **TryShowNext**
    
    - Проверка: CurrentWidget валиден? → Return.
        
    - Проверка: CinematicLock? → Return.
        
    - Если очередь пуста → Return.
        
    - Берём ID → удаляем из очереди.
        
    - Получаем данные из DataTable.
        
    - **Branch по CinematicLock**:
        
        - True → Create Widget (Mode = SkipHint).
            
        - False → Create Widget (Mode = Tutorial).
            
    - Сохраняем в переменную `CreatedWidget`.
        
    - Bind Event → OnPromptClosed → HandlePromptClosed.
        
    - Set CurrentWidget = CreatedWidget.
        
    - Add to Viewport.
        
    - Set PromptText из DataTable.
        
3. **CompletePrompt(Tag)**
    
    - Проверка: CurrentWidget валиден? → Return.
        
    - Берём DataRow по CurrentPromptID.
        
    - Сравниваем Tag → если совпадает → вызываем CurrentWidget → ClosePrompt.
        
4. **HandlePromptClosed**
    
    - CurrentWidget = None.
        
    - CompletedIDs += CurrentPromptID.
        
    - CurrentPromptID = None.
        
    - FindNextPrompt → если найден → ShowTutorial(NextID).
        

📌 Подсказка:  
**Manager – это диспетчер автобуса.** Он сам решает, кого пустить в салон (какая подсказка идёт дальше). Widget – это просто один пассажир.

---

## 🔹 Глава 4. Character (сенсоры игрока)

### 🎯 Цель

Фиксирует действия и сообщает Manager’у.

### Шаги

1. На BeginPlay → Get All Actors of Class (BP_BoxManager) → Set BoxManager.
    
2. Для Movement (IA_Move) → если длина вектора > 0.2 → CompletePrompt("Tutorial.Move").
    
3. Для Jump (IA_Jump) → первый прыжок → CompletePrompt("Tutorial.Jump").
    
4. Для Enter → если не кат-сцена → CompletePrompt("Skip").
    

📌 Подсказка:  
Character не рисует UI, не знает про анимации. Он только фиксирует действия игрока и «стучится» в BoxManager.

---

## 🔹 Глава 5. Level Blueprint (режиссёр)

### 🎯 Цель

Управляет кат-сценой и CinematicLock.

### Шаги

1. On BeginPlay
    
    - Найти BP_BoxManager → Set CinematicLock = true.
        
    - Play Sequence.
        
    - Bind Event OnFinished → OnFinished_Event.
        
2. OnFinished_Event
    
    - Set CinematicLock = false.
        
    - CompletePrompt("Skip").
        
    - TryShowNext.
        
    - Enable Input.
        
3. Обработчик Enter (во время кат-сцены)
    
    - Если CinematicLock = true → вызвать SkipCinematic (Stop Sequence, CinematicLock = false).
        
    - BoxManager → CompletePrompt("Skip").
        
    - Enable Input.
        

📌 Подсказка:  
Level Blueprint – это **режиссёр театра**. Он запускает кат-сцену, останавливает её и говорит BoxManager’у: «Теперь твой выход».

---

# 🏛 Общая схема архитектуры

`Character → CompletePrompt(Tag) → BoxManager → Create Widget → Widget  Level BP → CinematicLock/SkipCinematic ───────────────┘ ↑ Widget → OnPromptClosed → BoxManager → TryShowNext ─────┘`

---

## 🔹 Глава 6. Проверка сценариев

1. **Игрок ждёт кат-сцену**
    
    - Кат-сцена стартует → Greeting (SkipHint).
        
    - Кат-сцена завершается → SkipHint закрывается → Movement появляется.
        
2. **Игрок жмёт Enter во время кат-сцены**
    
    - SkipHint закрывается → кат-сцена останавливается → Movement появляется.
        
3. **Игрок выполняет действия**
    
    - Двигается → Movement закрывается → Jump.
        
    - Прыгает → Jump закрывается → Interact.
        

---

# 💡 Итоговая мудрость для новичка

- **Каждый Blueprint занимается только своим делом.**
    
- **Все связи идут через ссылки и Event Dispatchers.**
    
- **Level BP не создаёт виджеты напрямую.**
    
- **BoxManager не слушает клавиши.**
    
- **Widget не решает, что будет дальше.**