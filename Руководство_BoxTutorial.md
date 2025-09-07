## 🔹 Глава 1. Структура BP_BoxManager

### ⚜️ Функции

1. **ShowTutorial(ID : Name)**  
    Получает имя подсказки, кладёт его в очередь и пытается показать.
    
2. **TryShowNext()**  
    Проверяет, есть ли подсказка в очереди, достаёт данные из DataTable, создаёт и показывает виджет.
    
3. **CompletePrompt(Tag : Name)**  
    Завершает подсказку по тегу (вызывается из триггеров).
    
4. **FindNextPrompt()**  
    Находит следующую подсказку (по приоритету, исключая уже завершённые).
    
5. **HandlePromptClosed (Custom Event)**  
    Реакция на закрытие виджета: очистка текущего, добавление в CompletedIDs, запуск следующей подсказки.
    

---

### ⚜️ Переменные

1. **TutorialDataTable (Data Table)**  
    Ссылка на таблицу `DT_TutorialPromptsble`.
    
2. **PendingQueue (Array of Name)**  
    Очередь подсказок, ожидающих показа.
    
3. **CurrentPromptID (Name)**  
    ID текущей активной подсказки.
    
4. **CompletedIDs (Array of Name)**  
    Список уже завершённых подсказок.
    
5. **CurrentWidget (WBP_TutorialPrompt)**  
    Ссылка на активный виджет подсказки.
    
6. **CinematicLock (Bool)**  
    Флаг, запрещающий показ подсказок во время кат-сцен.
    
7. **BestID (Name)**  
    Временная переменная для хранения лучшего кандидата (внутри FindNextPrompt).
    
8. **BestPriority (Integer)**  
    Временная переменная для сравнения приоритетов (внутри FindNextPrompt).
## 🔹 Глава 2. EventGraph и функции

---

### ⚜️ EventGraph

1. **Event BeginPlay**
    
    - Сразу вызывает `ShowTutorial`, передавая ID `"Greeting"`.
        
    - Это гарантирует, что первой появится приветственная подсказка.
        
2. **HandlePromptClosed (Custom Event)**  
    Логика, которая срабатывает, когда закрывается текущая подсказка (по кнопке или по триггеру):
    
    - **Set CurrentWidget = None** → очищаем ссылку.
        
    - Проверяем, есть ли у нас реальный ID:
        
        - Узел `== None` + `NOT` + `Branch`.
            
        - Если `CurrentPromptID ≠ None` → значит, подсказка действительно существовала.
            
    - **Add(CurrentPromptID → CompletedIDs)** → добавляем ID в список завершённых.
        
    - **Set CurrentPromptID = None** → сбрасываем.
        
    - Вызываем `FindNextPrompt` → получаем NextID.
        
    - Если NextID ≠ None → вызываем `ShowTutorial(NextID)`.
        

---

### ⚜️ ShowTutorial (ID)

Логика добавления подсказки в очередь:

1. **Вход ShowTutorial(ID : Name)**
    
    - Получает имя подсказки.
        
2. **Добавляем ID в PendingQueue**
    
    - Узел **Add** → кладём ID в конец массива очереди.
        
3. **Вызов TryShowNext**
    
    - Сразу пробуем показать подсказку (если можно).
        

---

### ⚜️ TryShowNext()

Главная функция показа.

1. **Проверка: есть ли уже активный виджет**
    
    - Узел `Is Valid (Exec)` на `CurrentWidget`.
        
    - Если Valid → Return (подсказка уже показывается, новую не запускаем).
        
2. **Проверка: идёт ли кат-сцена**
    
    - Узел `Branch` → Condition = `CinematicLock`.
        
    - Если True → Return (не показываем).
        
3. **Проверка: есть ли что-то в очереди**
    
    - Узел `Length` на `PendingQueue`.
        
    - Если `Length == 0` → Return.
        
4. **Берём первую подсказку из очереди**
    
    - Узел `GET` → берём элемент с индексом 0.
        
    - `Set CurrentPromptID` = этот ID.
        
    - Узел `RemoveIndex(0)` → убираем из массива.
        
5. **Получаем данные из DataTable**
    
    - Узел `Get Data Table Row`, таблица = `TutorialDataTable`, RowName = `CurrentPromptID`.
        
    - Если строка не найдена → Print "СТРОКА НЕ НАЙДЕНА" и Return.
        
6. **Создаём виджет**
    
    - Узел `Create Widget`, класс = `WBP_TutorialPrompt`.
        
    - OwningPlayer = `Get Player Controller`.
        
    - Узел `Bind Event to OnPromptClosed` → создаём Event → `HandlePromptClosed`.
        
    - `Set CurrentWidget` = созданный виджет.
        
    - `Add to Viewport`.
        
7. **Передаём данные в виджет**
    
    - Узел `Break Struct (TutorialPrompt)` → ID, Text, Tag, Priority.
        
    - `Set PromptText` у CurrentWidget.
        
8. **Логирование**
    
    - Узел `Append` ("Подсказка " + ID).
        
    - `Print String`.
### ⚜️ CompletePrompt(Tag : Name)

Эта функция вызывается из триггеров (например, при движении, прыжке и т.д.), чтобы закрыть нужную подсказку.

1. **Проверка: есть ли текущий виджет**
    
    - Узел `Is Valid` → Input = `CurrentWidget`.
        
    - Если **Not Valid** → Return (подсказка не отображается).
        
2. **Получаем строку из DataTable**
    
    - Узел `Get Data Table Row` → Table = `TutorialDataTable`, RowName = `CurrentPromptID`.
        
    - Если Row Not Found → Return.
        
3. **Break Struct TutorialPrompt**
    
    - Получаем `ID`, `Text`, `CompletionTag`, `Repeatable`, `Priority`.
        
4. **Сравниваем CompletionTag**
    
    - Узел `==` → Tag (входной параметр) сравнивается с CompletionTag из строки.
        
    - Если не совпадает → Return.
        
5. **Закрываем подсказку**
    
    - Узел `ClosePrompt` → Target = `CurrentWidget`.
        
    - Это вызовет анимацию FadeOut и событие `HandlePromptClosed`.
        

---

### ⚜️ FindNextPrompt() → Name

Эта функция ищет следующую подсказку, которую нужно показать (минимальный Priority среди доступных).

1. **Подготовка**
    
    - Выходное значение: `NextID : Name`.
        
    - Изначально пустое.
        
    - `BestPriority = 9999` (очень большое число).
        
2. **Получаем все ID из DataTable**
    
    - Узел `Get Data Table Row Names`.
        
    - Это список всех строк таблицы.
        
3. **Цикл ForEach по строкам**  
    Для каждого ID:
    
    - Проверяем, есть ли ID в `CompletedIDs`.
        
    - Узел `Array Contains`.
        
    - Если ID уже завершён → Skip (Branch → False).
        
4. **Получаем данные строки**
    
    - `Get Data Table Row` для текущего ID.
        
    - Break Struct → берём Priority.
        
5. **Сравнение с BestPriority**
    
    - Узел `<` → Priority < BestPriority?
        
    - Если да → обновляем:
        
        - `BestPriority = Priority`.
            
        - `BestID = CurrentID`.
            
6. **Возвращаем результат**
    
    - После цикла `ReturnNode` → `NextID = BestID`.
        
    - Если ничего не найдено → вернётся None.
## 🔹 Глава 3. Виджет подсказки (WBP_TutorialPrompt)

Виджет отвечает за визуальное отображение подсказки и её плавное появление/исчезновение. Здесь реализована анимация, закрытие и связь с `BP_BoxManager`.

---

### ⚜️ Переменные и анимации

**Анимации (Animations):**

1. **FadeIn** → используется для плавного появления подсказки.
    
2. **FadeOut** → используется для плавного скрытия подсказки.
    

---

**Переменные (Variables):**

1. **PromptText (Text)** → текст подсказки, отображается в UI.
    
2. **DisplayTime (Float)** → время отображения подсказки (может задаваться менеджером).
    

---

**Event Dispatchers:**

1. **OnPromptClosed** → событие, уведомляющее `BP_BoxManager`, что подсказка завершена.
    

---

### ⚜️ Логика EventGraph

#### 1. FadeIn при показе

- Узел **Event Construct** запускает анимацию **FadeIn**.
    
- Логика: когда виджет создан, он плавно появляется.
    

---

#### 2. ClosePrompt (Закрытие подсказки)

- Кастомное событие **ClosePrompt**:
    
    - Запускает анимацию **FadeOut**.
        
    - После небольшой задержки (Delay 0.5) → **Remove from Parent** (удаляет виджет).
        
    - Далее вызывает **Call OnPromptClosed**, чтобы сообщить `BP_BoxManager`, что подсказка закрыта.
        

---

### ⚜️ Функция ApplyData

- Принимает параметры:
    
    - **In Text** (текст подсказки),
        
    - **In Display Time** (время отображения).
        
- Устанавливает значения в переменные **PromptText** и **DisplayTime**.
    
- Логика: позволяет менеджеру централизованно передавать данные в виджет.

# 📖 Глава 4. BP_ThirdPersonCharacter (Триггеры и связь с BoxManager)

Эта глава описывает, как в персонаже реализованы **триггеры подсказок** (движение, прыжок, пропуск Enter) и как он связывается с менеджером подсказок (**BP_BoxManager**).

---

## ⚜️ Переменные (My Blueprint)

1. **TutorialManagerRef (Object)** → временная ссылка (старое), сейчас используется `BoxManager`.
    
2. **HasMovedOnce (Boolean)** → проверка, сделал ли игрок шаг хотя бы один раз.
    
3. **HasJumpedOnce (Boolean)** → проверка, прыгнул ли игрок хотя бы один раз.
    
4. **BoxManager (BP_BoxManager Ref)** → ссылка на объект-менеджер, через него все триггеры управляют системой подсказок.
    

---

## ⚜️ Логика при старте игры

📌 Схема: **Event BeginPlay → Get All Actors of Class (BP_BoxManager) → Get (0) → Set BoxManager**

Описание:

- При запуске игры персонаж ищет **BoxManager** в уровне.
    
- Берётся **первый найденный экземпляр** (индекс [0]).
    
- Сохраняется в переменной **BoxManager**.
    
- Теперь все триггеры (движение, прыжок, пропуск) могут обращаться к системе подсказок.
    

---

## ⚜️ Триггер: Движение (Movement)

📌 Схема: EnhancedInputAction → Make Vector2D → Vector2D Length → > 0.2 → Branch → Проверка ID → CompletePrompt.

Описание:

1. **EnhancedInputAction_IA_Move** получает X и Y.
    
2. Эти значения собираются в **Vector2D**, затем вычисляется его длина.
    
3. Если длина > 0.2 → значит, игрок действительно двинулся.
    
4. Проверка: текущая подсказка действительно **Movement**?
    
    - Если да → вызываем `CompletePrompt` с тегом **Tutorial.Move**.
        
5. Подсказка закрывается, и менеджер показывает следующую (Jump).
    

---

## ⚜️ Триггер: Прыжок (Jump)

📌 Схема: EnhancedInputAction_IA_Jump → Branch (HasJumpedOnce = false) → Set HasJumpedOnce = true → CompletePrompt(Tutorial.Jump).

Описание:

1. Игрок нажимает клавишу прыжка (IA_Jump).
    
2. Проверка: прыгал ли он уже раньше?
    
    - Если нет → выполняется триггер.
        
    - Если да → повторно не сработает.
        
3. Устанавливаем `HasJumpedOnce = true`.
    
4. Вызываем `CompletePrompt` с тегом **Tutorial.Jump**.
    
5. Подсказка прыжка закрывается, менеджер открывает следующую.
    

---

## ⚜️ Триггер: Пропуск подсказки (Skip)

📌 Схема: InputKey (Enter) → Get All Actors of Class (BP_BoxManager) → CompletePrompt(Skip).

Описание:

- Если игрок не хочет ждать или читать подсказку → он может нажать **Enter**.
    
- Вызов `CompletePrompt` с тегом **Skip** закрывает текущую подсказку.
    
- Менеджер сразу переходит к следующей.
    

---

## ⚜️ Итог работы Character

- **BP_ThirdPersonCharacter** не хранит логику показа подсказок.
    
- Его задача — **следить за действиями игрока (движение, прыжок, Enter)** и **сообщать BoxManager**, что соответствующая подсказка выполнена.
    
- Всё управление порядком подсказок и очередью остаётся за **BP_BoxManager**.
    

---

⚜️ Таким образом, **Character → BoxManager → Widget** образуют связку:

- Character фиксирует действие.
    
- BoxManager решает, какая подсказка следующая.
    
- Widget показывает её игроку.
# 📖 Глава 5. Level Blueprint (кат-сцена и синхронизация с подсказками)

Эта глава описывает, как уровень управляет **кат-сценой** и координирует её с системой подсказок, а также как добавлен механизм пропуска кат-сцены по кнопке **Enter**.

---

## ⚜️ Основные задачи Level Blueprint

1. **Запуск кат-сцены** при старте уровня.
    
2. **Установка флага CinematicLock** в `BoxManager`, чтобы во время кат-сцены не появлялись подсказки.
    
3. **Отслеживание завершения кат-сцены** и снятие блокировки.
    
4. **Возможность пропустить кат-сцену по кнопке Enter** с плавным затемнением/осветлением экрана.
    
5. **Передача управления игроку** и запуск подсказок.
    

---

## ⚜️ Схема работы

### 1. При старте уровня

- **Event BeginPlay** → `Get All Actors of Class (BP_BoxManager)` → `Set URLBoxManager` → `Set CinematicLock = true`.  
    ⚡ Это блокирует показ подсказок до завершения кат-сцены.
    
- Одновременно создаётся `Level Sequence Player` для кат-сцены (`Start_scene`).
    
- Переменная `SequencePlayerRef` сохраняет ссылку на проигрыватель, чтобы позже можно было его остановить.
    

### 2. Запуск кат-сцены

- Вызов `Play` у **Movie Scene Sequence Player (Start_scene)`.
    
- Перед этим выполняется `Bind Event to OnFinished`, который подключает наш кастомный эвент `OnFinished_Event`.  
    ⚡ Благодаря этому, когда кат-сцена завершится, автоматически вызовется нужная логика.
    

### 3. Конец кат-сцены (OnFinished_Event)

- Вызывает кастомный эвент `SkipCinematic`, который делает следующее:
    
    1. `Stop` у SequencePlayerRef.
        
    2. Снимает `CinematicLock = false` в `BoxManager`.
        
    3. Запускает `TryShowNext`, чтобы система подсказок начала работу.
        
    4. Возвращает управление игроку через `Set View Target with Blend`.
        
    5. Выполняет `Enable Input (PlayerCharacter)`.
        
    6. Выполняет `Disable Input (Level Blueprint)`, чтобы Enter больше не перехватывался уровнем.
        

### 4. Пропуск кат-сцены (Enter)

- В Level Blueprint добавлен обработчик кнопки **Enter**:
    
    1. `InputKey (Enter)` → `Get All Actors of Class (BP_BoxManager)`.
        
    2. Проверяется переменная `CinematicLock` через `Branch`.
        
    3. Если `CinematicLock = true` → выполняется пропуск:
        
        - Создаётся виджет **WBP_SkipFadeOut**.
            
        - Он запускает FadeIn (экран затемняется).
            
        - После небольшой задержки (0.5 сек) вызывается `SkipCinematic`.
            
        - Далее у виджета вызывается `StartFadeOut` (экран плавно возвращается в норму).
            
    4. Если `CinematicLock = false` → Enter игнорируется (теперь работает только в Character BP для пропуска подсказок).
        

### 5. Передача управления игроку

- После кат-сцены или её пропуска:
    
    - `Set View Target with Blend` возвращает камеру игрока.
        
    - `Enable Input (PlayerCharacter)` включает управление.
        
    - `Disable Input (Level BP)` отключает обработку Enter на уровне.
        

---

## ⚜️ Итог работы Level Blueprint

- При старте игры игрок **смотрит кат-сцену**, подсказки не мешают.
    
- Кат-сцену можно либо досмотреть до конца, либо пропустить нажатием Enter.
    
- Если нажать Enter → экран плавно затемняется, кат-сцена прерывается, экран плавно возвращается.
    
- После кат-сцены управление возвращается игроку.
    
- Менеджер подсказок (BoxManager) автоматически начинает показывать **Greeting** и дальше ведёт обучение.
    
- Кнопка Enter после кат-сцены остаётся только для пропуска подсказок.
---

Теперь у нас есть **полная структура из 5 глав**:

1. Общее устройство системы.
    
2. BP_BoxManager (функции и переменные).
    
3. WBP_TutorialPrompt (виджет).
    
4. BP_ThirdPersonCharacter (триггеры).
    
5. Level Blueprint (кат-сцена и синхронизация).