### 🎯 Назначение

`BP_CinemaManager` управляет кат-сценами и дирижирует подсказками через BoxManager.

---

## 🔹 Переменные

(как ранее)

1. **CinemaData (Data Table)** — таблица кат-сцен.
    
2. **CurrentCinemaID (Name)** — ID активной сцены.
    
3. **SequenceActorRef (Level Sequence Actor Ref)** — Actor для сцены.
    
4. **SequencePlayer (Level Sequence Player Ref)** — Player для сцены.
    
5. **CinematicLock (Bool)** — идёт ли кат-сцена.
    
6. **CompletedCinemas (Name Array)** — просмотренные сцены в текущей сессии.
    

---

## 🔹 Функции

1. **PlayCinema(ID : Name)**
    
    - Проверяет `CompletedCinemas` и SaveGame (`WatchedCinemas`).
        
    - Если сцена уже просмотрена → `Return`.
        
    - Если новая:
        
        - `CinematicLock = true`.
            
        - Вызывает BoxManager → `ShowSkipHint`.
            
        - Создаёт Player (Create Level Sequence Player).
            
        - Подписывается на `OnFinished`.
            
        - Запускает `Play`.
            
2. **HandleCinemaFinished()**
    
    - Снимает `CinematicLock = false`.
        
    - Добавляет `CurrentCinemaID` в `CompletedCinemas` и `WatchedCinemas` (SaveGame).
        
    - Вызывает:
        
        - BoxManager → `HideSkipHint`.
            
        - BoxManager → `ShowTutorial("Greeting")` (или другой ID, если очередь уже заполнена).
            
3. **SkipCinema()**
    
    - Проверяет `SequencePlayer`.
        
    - Если валиден → `Stop`.
        
    - Затем вызывает `HandleCinemaFinished`.
        

---

## 🔹 События

1. **OnFinished_Event (Custom Event)**
    
    - Срабатывает после завершения Level Sequence (из Bind On Finished).
        
    - Вызывает `HandleCinemaFinished`.
        
2. **BeginPlay (Event)**
    
    - Может быть использован для инициализации (например, загрузка SaveGame или подготовка CompletedCinemas).
        

---

## 🔹 Архитектура

- CinemaManager теперь не только проигрывает сцены, но и сообщает BoxManager, что показывать.
    
- Он управляет SkipHint и Tutorial по событиям.
    
- Гарантирует, что сцены не повторяются.