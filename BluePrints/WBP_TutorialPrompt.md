# WBP_TutorialPrompt — описание архитектуры

### 🎯 Назначение

`WBP_TutorialPrompt` отвечает за **отображение текста подсказок и окна SkipHint во время кат-сцены**.  
Он универсален: в зависимости от режима (`E_TutorialMode`) показывает либо **обычное tutorial-сообщение**, либо **подсказку "Нажмите Enter, чтобы пропустить"**.

---

## 🔹 Переменные

1. **Animations**
    
    - `FadeIn` → анимация плавного появления.
        
    - `FadeOut` → анимация плавного исчезновения.
        
2. **TutorialGroup (Widget Reference)**
    
    - Контейнер UI-элементов для стандартных Tutorial-подсказок.
        
3. **SkipHintGroup (Widget Reference)**
    
    - Контейнер UI-элементов для режима SkipHint.
        
4. **T_Prompt (TextBlock)**
    
    - Основной текст Tutorial-подсказки.
        
    - Связан с переменной `PromptText`.
        
5. **SkipHintText (TextBlock)**
    
    - Текст для режима SkipHint:
        
        `«Нажмите Enter, чтобы пропустить сцену»`
        
6. **PromptText (Text)**
    
    - Данные, которые передаёт BoxManager.
        
    - Используется для отображения Tutorial-подсказки.
        
7. **PromptRef (Object)**
    
    - Резервная переменная для хранения ссылки на источник данных (необязательная).
        
8. **AutoFadeOut (Boolean)**
    
    - Флаг, нужно ли автоматически скрыть подсказку через время.
        
9. **AutoFadeTime (Float)**
    
    - Время отображения подсказки перед автозакрытием.
        
10. **Mode (Enum E_TutorialMode)**
    

- Режим работы виджета:
    
    - `Tutorial` → отображается TutorialGroup.
        
    - `SkipHint` → отображается SkipHintGroup.
        

---

## 🔹 События

1. **Event Construct**
    
    - При создании виджета выполняет проверку `Mode`.
        
    - Узел `Switch on E_TutorialMode`:
        
        - Если `Tutorial` → делает видимым `TutorialGroup`, скрывает `SkipHintGroup`.
            
        - Если `SkipHint` → наоборот.
            
    - Запускает анимацию **FadeIn**.
        
2. **ClosePrompt (Custom Event)**
    
    - Запускает анимацию **FadeOut**.
        
    - После задержки (`Delay 0.5`) вызывает **Call OnPromptClosed**, уведомляя `BoxManager`, что подсказка завершена.
        
    - Затем удаляет себя с экрана (`Remove from Parent`).
        

---

## 🔹 Функции

1. **ApplyData(In Text)**
    
    - Вход: `In Text (Text)`.
        
    - Устанавливает значение в переменную `PromptText`.
        
    - Используется BoxManager для централизованной передачи данных в виджет.
        

---

## 🔹 Event Dispatcher

1. **OnPromptClosed**
    
    - Событие для связи с `BP_BoxManager`.
        
    - Гарантирует, что после закрытия виджета Manager обновит очередь подсказок.
        

---

## 🔹 UI-элементы (Designer)

- **TutorialGroup** → содержит:
    
    - `T_Prompt` (TextBlock, крупный шрифт, по центру).
        
- **SkipHintGroup** → содержит:
    
    - `SkipHintText` (TextBlock, текст: «Нажмите Enter, чтобы пропустить сцену»).
        
- **Анимации:**
    
    - `FadeIn` → плавное появление обеих групп.
        
    - `FadeOut` → плавное исчезновение перед удалением.
        

---

## 🔹 Итоговая роль в архитектуре

`WBP_TutorialPrompt` является **универсальным виджетом подсказок**.

- В режиме `Tutorial` показывает обычный текстовый хинт.
    
- В режиме `SkipHint` выводит подсказку для кат-сцены (Enter для пропуска).
    
- Работает в связке с `BP_BoxManager`, который решает **какой режим активен** и **какие данные передать**.