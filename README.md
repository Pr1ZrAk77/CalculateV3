# Module-1,2,3

Этот проект реализует веб-сервис, принимающий выражение через Http запрос и возвращабщий результат вычислений,имеющий регистарцию и хранение данных пользователей.

Инструкция по запуску:

1)Убедитесь, что у вас установлен Go (версия 1.16 или выше).

2)Убедитесь, что у вас установлен SQLite3.

3)включите CGO (если выключен) командой export CGO_ENABLED=1.

4)скачайте gcc compiler.

6)Для работы фронтэнда скачайте Node.js на сайте (https://nodejs.org/en/download)

7)Скопируйте репозиторий(через git bash):

```bash
git clone https://github.com/Pr1ZrAk77/CalculateV3
```

```bash
cd CalculateV3
```

Запускаем orchestator:

```bash
export ORCHESTRATOR_URL=localhost:50051
export TIME_ADDITION_MS=200
export TIME_SUBTRACTION_MS=200
export TIME_MULTIPLICATIONS_MS=300
export TIME_DIVISIONS_MS=400

go run cmd/orchestrator.start/main.go
```
Затем запускаем agent:

```bash
export COMPUTING_POWER=4
export ORCHESTRATOR_URL=localhost:50051

 go run cmd/agent.start/main.go
```

Регестрируем нового пользователя:

```bash
curl -X POST http://localhost:8080/api/v1/register \
  -H "Content-Type: application/json" \
  -d '{"login":"user1","password":"password123"}'
```

Входим как пользователь:

```bash
curl -X POST http://localhost:8080/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"login":"user1","password":"password123"}'
```

далее при запуске надо будет использовать свой токен 

```bash
curl --location 'http://localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_JWT_TOKEN' \
--data '{"expression": "2+2*2"}'
```

Примеры использования:

Успешный запрос:

```bash
curl --location 'http://localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_JWT_TOKEN' \
--data '{"expression": "2+2*2"}'
```

Ответ:

```bash
{
  "id": "..."
}
```

После можно посмотреть этап выполнения данного запроса и его результат(если уже вычислилось ):

```bash
curl --location 'http://localhost:8080/api/v1/expressions' \
--header 'Authorization: Bearer YOUR_JWT_TOKEN'
```

Вывод:

```bash
{"expressions":[{"id":"1","expression":"2*2+2,"status":"pending"}]}
```

Если вычисления выполнены то:

```bash
{"expression":{"id":"1","expression":"2*2+2","status":"completed","result":6}}
```

Ошибки при запросах:

Ошибка при создании пользователя который уже существует:

```bash
{"error":"User already exists"}
```

Ошибка 404(отсутствие выражения ):

```bash
{"error":"API Not Found"}
```

Ошибка 422 (невалидное выражение ):

```bash
curl --location 'http://localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_JWT_TOKEN' \
--data '
{
  "expression": "2+a"
}'

```
Ответ:

```bash
{
  {"error":"expected number at position 2"}
}
```

Ошибка неправильного знака:

```bash
curl --location 'http://localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_JWT_TOKEN' \
--data '
{
  "expression": "2\0"
}'
```

Ответ:

```bash
{"error":"Invalid token"}
```

Ошибка 500 (внутренняя ошибка сервера ):

```bash
curl --location 'http://localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '
{
  "expression": "2/0"
}'
```
Ответ(у  меня высвечивается изначально id созданной задачи,а после в git bash где был запущен agent.start можно увидеть что выводится деление на 0 ):

```bash
{
  Worker: error computing task : division by zero
}
```

Тесты запускаются тоже через git bash(или можно через visual studio code):

1)Сначала опять переходим в папку с модулем.

```bash
cd CalculateV3
```

2)Затем запускаем тестирование:

```bash
go test ./internal/agent/agent_calculation_test.go
```

Или

```bash
go test ./cmd/internal_test.go (интеграционный)
```

Или

```bash
go test ./internal/storage/storage_test.go
```

3)При успешном прохождение теста должен вывестись ответ:

```bash
ok  	calc_service/internal/evaluator	0.001s
```

4)При ошибке в тестах будет указано где она совершена.
P.S ошибка в тесте агента связанная с не указанным ErrDivivsionByZero появляется так как в функции тестирования я ее не оглашаю,
она создает конфликты в visual studio code так как уже присутствует в самом агенте