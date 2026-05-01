Правила работы с проектом karma-diary-bot:

1. МИНИМАЛЬНЫЕ ИЗМЕНЕНИЯ — меняй только то что просят. Строку текста → замени строку, не создавай функцию. Одно значение → одно значение.
2. НЕ ТРОГАЙ то что работает — не рефактори, не добавляй типы, комменты, docstrings к чужому коду.
3. ТЕКСТЫ — только в texts.py, но если это замена одной строки в scheduler/handler — просто замени строку.
4. ДЕПЛОЙ — после каждого фикса: git add → commit → git push origin main. Потом railway up --detach из /Users/mac/Downloads/Карма-дневнк. Проверить railway logs --tail 15.
5. БД — PostgreSQL через asyncpg (Supabase). DATABASE_URL в .env.
6. Railway проект — karma-diary-bot, сервис karma-diary-bot. railway status/logs ТОЛЬКО из корня /Users/mac/Downloads/Карма-дневнк.
