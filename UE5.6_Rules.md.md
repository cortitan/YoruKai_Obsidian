# Правила работы с Unreal Engine 5.6

1. В функциях нельзя использовать Delay и Add Custom Event.
2. Для анимаций используем Timeline или Widget Animation.
3. Всегда проверяем Anchors при создании UI.
4. Если используется Level Sequence — он должен быть размещён как Actor на уровне.
5. При работе с переменными и событиями указывать типы явно.
6. Проверять, что Event BeginPlay находится только в Level Blueprint или Character BP, но не в функциях.
