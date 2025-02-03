# TTSaveAPI: Документация

**TTSaveAPI** — это мощный инструмент для скачивания и стриминга медиа с TikTok. API поддерживает обработку URL, скачивание контента и обработку ошибок.

## Внимание
Для упрощённого пользования API рекомендуем использовать Python библиотеку [PyTTSave](https://github.com/FlacSy/PyTTSave).

## Базовая информация
-	**Базовый URL**:
```
http://45.13.225.104:4347/api/v3/
```

-	Версия API: v3
-	Формат ответов: JSON

## Эндпоинты

#### 1. POST /download

Скачивает медиа по указанному URL TikTok и возвращает **multipart stream**, содержащий одно или несколько файлов.

Тело запроса:
| Параметр  | Тип	| Обязательный | Описание |
| ------------- | ------------- | ------------- | ------------- |
| url  | string | да  | URL видео или фото из TikTok. |
| download_content_type  | string | да | Тип контента: Тип контента: **VIDEO_ONLY** (только видео, где коллажи будут присланы в виде видео) или **ORIGINAL** (исходный контент, включая фото, аудио, видео).|


Пример запроса:
```http
POST /download HTTP/1.1
Content-Type: application/json

{
  "url": "https://www.tiktok.com/@author123/video/312132123123132",
  "download_content_type": "VIDEO_ONLY"
}
```

Формат ответа

Сервер возвращает multipart stream (multipart/x-mixed-replace), содержащий один или несколько файлов. Каждый файл разделяется границей (boundary), а внутри блока указывается его Content-Type и Content-Disposition.

Пример ответа:
```http
HTTP/1.1 200 OK
Content-Type: multipart/x-mixed-replace; boundary=simple_boundary
Content-Meta: "Encoded Base64 json string"

--simple_boundary
Content-Type: video/mp4
Content-Disposition: attachment; filename="video.mp4"

(binary data)

--simple_boundary
Content-Type: image/jpeg
Content-Disposition: attachment; filename="image.jpg"

(binary data)

--simple_boundary
Content-Type: audio/mpeg
Content-Disposition: attachment; filename="audio.mp3"

(binary data)

--simple_boundary--
```
**Content-Meta** хранит в себе зашифрованный JSON посредством использования шифра Base64.

Пример расшифрованной Content-Meta:
```json
{"title": "Title", "desc": "Description"}
```
Коды ошибок:
-	400: Некорректный запрос (отсутствует url).

```json
{ "detail": "URL is required" }
```

-	400: Некорректный запрос (отсутствует или неправильный тип контента).

```json
{ "detail": "Invalid content type }
```

-	400: Если серверу не удалось скачать контент.

```json
{ "detail": "No files available for download" }
```

- 500: Ошибка сервера (например, не удалось скачать файл).
Пример ответа:

```json
{ "detail": "Failed to save the file" }
```

#### 2. POST /ping

Проверка работоспособности сервера.

Пример запроса:
```http
POST /ping HTTP/1.1
```
Ответ:
```
pong
```
## Обработка ошибок

TTSaveAPI имеет встроенные обработчики ошибок, чтобы гарантировать ясность ответов при возникновении проблем.

### Типы ошибок:

### 1. Глобальные ошибки

Возникают при неожиданных проблемах.

Пример ответа:
```json
{
  "message": "Internal Server Error",
  "detail": "Error: <подробности ошибки>"
}
```
#### 2. Ошибки валидации

Возникают при некорректных или отсутствующих параметрах в запросе.

Пример ответа:
```json
{
  "message": "Validation Error",
  "detail": [
    {
      "loc": ["body", "url"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```
## Как начать использовать API
#### 1.	Скачивание медиа:
	1.	При получении ответа клиент должен обработать поток multipart/x-mixed-replace, читая данные по boundary.
	2.	Каждый файл содержит заголовки Content-Type и Content-Disposition, которые помогают определить тип файла и его имя.
	3.	Парсинг потока возможен с использованием стандартных HTTP-библиотек. В Python, например, можно использовать requests с iter_content():
#### 2.	Обработка multipart stream:
Пример кода:
```python
boundary = b"--simple_boundary"
for chunk in response.iter_content(chunk_size=1024):
    if boundary in chunk:
        print("New file received!")
    else:
        with open("downloaded_file.mp4", "ab") as f:
            f.write(chunk)
```
#### 3.	Проверка сервера:
Отправьте запрос на /ping, чтобы проверить доступность API.
