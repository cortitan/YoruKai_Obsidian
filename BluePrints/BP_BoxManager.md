### 🎯 Назначение

`BP_BoxManager` управляет **подсказками (tutorial prompts)** и теперь полностью подчиняется сигналам от `BP_CinemaManager`.

- Сам больше не проверяет `CinematicLock`.
    
- Работает только по событиям: `ShowSkipHint`, `HideSkipHint`, `ShowTutorial`, `ShowNextTutorial`.
    

---

## 🔹 Переменные

1. **TutorialDataTable (Data Table)** — таблица подсказок.
    
2. **PendingQueue (Name Array)** — очередь подсказок.
    
3. **CurrentPromptID (Name)** — активный ID подсказки.
    
4. **CompletedIDs (Name Array)** — завершённые подсказки.
    
5. **CurrentWidget (WBP_TutorialPrompt Ref)** — текущий виджет.
    
6. **PromptMode (Enum E_TutorialMode)** — режим работы (`Tutorial` или `SkipHint`).
    

---

## 🔹 Функции и события

1. **ShowTutorial(ID : Name)**
    
    - Добавляет ID в очередь.
        
    - Вызывает `TryShowNext`.
        
2. **TryShowNext()**
    
    - Проверяет очередь.
        
    - Если есть ID → достаёт строку из DataTable.
        
    - Создаёт **Tutorial-виджет** (`Mode = Tutorial`).
        
    - Сохраняет в `CurrentWidget`.
        
    - Делает `Bind Event to OnPromptClosed`.
        
3. **ShowSkipHint (Custom Event)**
    
    - Создаёт **SkipHint-виджет** (`Mode = SkipHint`).
        
    - `Add to Viewport`.
        
    - Сохраняет в `CurrentWidget`.
        
4. **HideSkipHint (Custom Event)**
    
    - Удаляет `CurrentWidget` из вьюпорта.
        
    - Сбрасывает переменную `CurrentWidget = None`.
        
5. **ShowNextTutorial (Custom Event)**
    
    - Просто вызывает `TryShowNext`.
        
6. **CompletePrompt(Tag : Name)**
    
    - Завершает текущую подсказку.
        
7. **HandlePromptClosed (Custom Event)**
    
    - Сбрасывает `CurrentWidget`.
        
    - Добавляет ID в `CompletedIDs`.
        
    - Запускает `FindNextPrompt`.
        

---

## 🔹 Архитектура

BoxManager больше не решает сам, когда показывать SkipHint или Tutorial.  
Теперь он полностью ждёт команды от CinemaManager:

- `ShowSkipHint` → показать кнопку Enter.
    
- `HideSkipHint` + `ShowNextTutorial` → переключиться обратно на Tutorial.