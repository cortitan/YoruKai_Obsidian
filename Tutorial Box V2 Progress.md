📜 **Tutorial Box V2 — текущее состояние (обновлённое)**

🔹 **Переменные в BP_BoxManager**

- TutorialDataTable (Data Table) → ссылка на DT_TutorialPromptsble.
    
- PendingQueue (Array of Name) → очередь подсказок.
    
- CurrentPromptID (Name) → ID текущей подсказки.
    
- CompletedIDs (Array of Name) → список завершённых подсказок.
    
- CurrentWidget (WBP_TutorialPrompt, Single) → активный виджет подсказки.
    
- CinematicLock (Bool) → блокирует подсказки во время кат-сцены.
    

---

🔹 **Реализованные функции**

**ShowTutorial(ID)**

- Добавляет ID в очередь.
    
- Вызывает TryShowNext.
    

**TryShowNext()**

- Проверяет, показывается ли уже подсказка.
    
- Проверяет, не пустая ли очередь.
    
- Берёт первый ID из очереди.
    
- Достаёт данные из DataTable.
    
- Создаёт виджет, добавляет на экран.
    
- Передаёт в него данные из DataTable.
    
- Подписывает OnPromptClosed на HandlePromptClosed.
    
- Логирует успех.
    

**HandlePromptClosed (EventGraph)**

- Set CurrentWidget = None.
    
- ✅ Проверяет, что CurrentPromptID не пуст (Equal(Name) → NOT → Branch).
    
- ✅ Если есть значение → добавляет CurrentPromptID в CompletedIDs.
    
- ✅ После этого сбрасывает CurrentPromptID = None.
    
- Вызывает FindNextPrompt, а затем ShowTutorial (если есть новая подсказка).
    

**FindNextPrompt()**

- Получает все ID из DataTable.
    
- Пробегает цикл по строкам.
    
- Проверяет: не был ли ID в CompletedIDs.
    
- Если подсказка новая → достаёт строку.
    
- Сравнивает её Priority с текущим BestPriority.
    
- Сохраняет ID с наименьшим приоритетом как NextID.
    
- Возвращает найденный NextID.
    

---

🔹 **Виджет WBP_TutorialPrompt**

- FadeIn (анимация при появлении).
    
- FadeOut (анимация при скрытии).
    
- ClosePrompt() → проигрывает FadeOut, удаляет виджет, вызывает OnPromptClosed.
    
- OnPromptClosed (Event Dispatcher) → уведомляет Manager о закрытии.
    

---

🔹 **DataTable (DT_TutorialPromptsble)**  
Колонки:

- ID (Name).
    
- Text (FText).
    
- CompletionTag (Name) → внешний триггер для закрытия.
    
- Repeatable (Bool).
    
- Priority (Int).
    

---

🔹 **Готово ✅**

- Очередь и показ подсказок.
    
- FadeIn и FadeOut.
    
- Закрытие подсказки по событию.
    
- Manager переключает на следующую подсказку.
    
- CinematicLock блокирует показ во время кат-сцены.
    
- FindNextPrompt находит подсказку с минимальным Priority.
    
- ⚡ Исправлено: HandlePromptClosed теперь добавляет CurrentPromptID в CompletedIDs (через проверку Equal(Name) → NOT).
    
- Подсказки больше не повторяются.