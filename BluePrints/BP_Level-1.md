### 🎯 Назначение

Level BP отвечает только за:

- загрузку SaveGame,
    
- проверку прогресса,
    
- запуск CinemaManager.
    

---

## 🔹 Переменные

- Save Slot Name = `"MainSave"`.
    

---

## 🔹 Логика

1. **Event BeginPlay**
    
    - Загружает SaveGame.
        
    - Проверяет `TutorialCompleted`.
        
2. **Если TutorialCompleted = True**
    
    - Запускается только кат-сцена (CinemaManager).
        
3. **Если TutorialCompleted = False**
    
    - Запускается только CinemaManager.
        
    - ⚠️ Вызов `BoxManager.ShowTutorial("Greeting")` убран.
        

---

## 🔹 Архитектура

Level Blueprint больше не управляет подсказками.  
Теперь он запускает только CinemaManager → а тот сам вызывает BoxManager.