# Pyth
    import argparse
    import urllib.request
    import os
    import sys
    from html.parser import HTMLParser
    import json
    
    CACHE_FILE = 'cache.json' #Подключение кэша

# Определяем класс парсера HTML
    class MyHTMLParser(HTMLParser):
        def __init__(self):
            super().__init__()
            self.headers = []
        def handle_starttag(self, tag, attrs):
            if tag.startswith('h') and len(tag) == 2 and tag[1] in '123456':
                self.headers.append(tag)
        def handle_data(self, data):
            if self.headers:
                self.headers[-1] += f" {data}"
    
    # Функция для загрузки HTML-страницы
    def fetch_html(url):
        try:
            with urllib.request.urlopen(url) as response:
                return response.read().decode('utf-8')
        except Exception as e: #e - переменная, которая используется для перехвата и обработки ошибок в модуле urllib
            print(f"Ошибка загрузки страницы: {e}")
            sys.exit(1)

# Функция для парсинга HTML и извлечения заголовков
    def parse_html(html):
        parser = MyHTMLParser()
        parser.feed(html)
        return parser.headers

# Чтение файла для кэширования, проверка существования
    def load_cache():
        if os.path.exists(CACHE_FILE):
            with open(CACHE_FILE, 'r') as f: #f - объект файла
                return json.load(f)
        return {}

# Запись заголовков для указанного  url в кэш
    def save_cache(url, headers):
        cache = load_cache()
        cache[url] = headers
        with open(CACHE_FILE, 'w') as f:
            json.dump(cache, f)

# Основная функция
    def main():
        parser = argparse.ArgumentParser(description="Загрузить HTML и извлечь заголовки.") # Создание парсера
        parser.add_argument("url", help="URL страницы для загрузки") # Добавляем url в качестве аргумента     
        args = parser.parse_args() # Сохранение полученных данных от пользователя 
        url = args.url #Доступ к значению url
        # Проверяем, что URL начинается с http или https
        if not (url.startswith("http://") or url.startswith("https://")):
            print("Пожалуйста, укажите корректный URL.")
            sys.exit(1)
  # Кэшируем
    cache = load_cache()
    if url in cache:
        headers = cache[url]
        print("Использование кэша:")
    else:
        html = fetch_html(url)
        headers = parse_html(html)
        save_cache(url, headers)

    html = fetch_html(url)
    headers = parse_html(html)

    print("Найденные заголовки:")
    for header in headers:
        print(header)
# Заупскаем основную функцию
    if __name__ == "__main__":
        main()




 
