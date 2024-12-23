# Кастомизация статического сайта

### Задание

На основе результатов выполнения предыдущей работы, кастомизируйте свой собственный сайт, разобравшись в том как возможно использовать свою собственную тему (шаблон для сайта) и создать свой шаблон (на занятии был продемонстрирован пример создания темы и шаблонизации с помощью Jinja2 для mkdocs), выполните сборку сайта с помощью GitHub Actions.

Задание:

1. Создать собственную тему на основе HTML, CSS, JS с использованием или без использования CSS-библиотек таких как Bootstrap, Bulma, фреймворков (например, Tailwind), JS-библиотек для разработки фронтэнда (например, React).
2. На основе репозитория разработать пайплайн или набор пайплайнов (yml-файл) для тестирования и сборки статики (HTML, CSS, JS) — фронтэнда сайта, а затем построения собственно самого сайта (интеграции контента в разметке Markdown в шаблон сайта) и деплоя его на GitHub Pages.
3. Необходимо учесть, что HTML файлы должны валидироваться на корректность, минифицироваться, должна быть предусмотрена сборка с помощью PostCSS (см. репозиторий).
4. Дополнительным этапом (необязательным) реализуйте улучшение типографики содержимого сайта (например, используя этот инструмент).

Требования:

1. Минимальные требования к теме: наличие кастомного header, footer, а также стилизованной страницы для одной из секций (например, главной страницы).
2. Обязательно добавьте метаданные сайта (название, описание, автор) (тег meta).
3. Сайт должен быть доступен по адресу GitHub Pages. Убедитесь, что все страницы и стили корректно отображаются.

В качестве ответа - ссылка на GitHub Pages-сайт (и на репозиторий, если открыт), текстовый отчет (README.md в репозитории или страница на сайте) с демонстрацией выполненных действий для развертывания, содержимым yml-файла, описанием этапов сборки и т. д..

### Выполнение

1. Создаем папку `my-theme`, в которой будут находиться все файл шаблоны, в themes
   структура папки my-theme

2. Добавляем разметку для `baseof.html`
   ```
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <meta name="description" content="{{ .Site.Params.description }}">
       <meta name="author" content="{{ .Site.Params.author }}">
       <title>{{ .Site.Title }}</title>
       <link rel="stylesheet" href="css/style.css">
       {{ block "head" . }}{{ end }}
   </head>
   <body class="page">
       <hr>
       {{ block "header" . }}{{ end }}
       <main role="main">
       {{ block "main" . }}{{ end }}
       </main>
       <hr>
       {{ block "footer" . }}{{ end }}
   </html>
   ```
3. Добавляем разметку для `index.html`
   ```
   {{ define "main" }}
       <section class="content">
           <div class="container">
           <h2>Latest Posts</h2>
           <div class="posts">
               {{ range first 10 .Site.RegularPages }}
               <a href="{{ .Permalink }}" class="read-more">
               <div class="post-preview">
                   <p>{{ .Summary }}</p>
               </div>
               </a>
               {{ end }}
           </div>
           </div>
       </section>
   {{ end }}
   ```
4. Добавляем разметку для `footer.html`

   ```
   {{ block "footer" . }}
       <footer>
           <p>&copy; 2024 {{ .Site.Params.author }}</p>
       </footer>
   {{ end }}

   ```

5. Добавляем разметку для `header.html`
   ```
   {{ block "header" . }}
       <header>
           <h1>{{ .Site.Title }}</h1>
       </header>
   {{ end }}
   ```
6. Добавляем разметку для `single.html`

   ```
   {{ define "main" }}
      <article>
          <header class="post__header">
          <h2 class="post__title">{{ .Title }}</h2>
          <p class="post__date"><small>{{ .Date.Format "January 2, 2006" }}</small></p>
          </header>
          <section class="post__content">
          {{ .Content }}
          </section>
      </article>
   {{ end }}
   ```

7. Создаем файл `postcss.congig.js`
   ```
   module.exports = {
       plugins: [
           require('autoprefixer'),
           require('cssnano')
       ]
   };
   ```
8. Создаем скрипт для запуск `postcss` в `package.json`
   ```
   "scripts": {
       "build:css": "postcss themes/my-theme/static/css/*.css -d static/css/"
   },
   ```
9. Создаем `yml` файл для автоматизации деплоя

   ```
   name: Build and Deploy Hugo Site

   on:
   push:
       branches:
       - main
       pull_request:
   jobs:
   deploy:
       runs-on: ubuntu-22.04
       steps:
       - name: Checkout code
       uses: actions/checkout@v3

       - name: Set up Node.js
       uses: actions/setup-node@v3
       with:
           node-version: '20'

       - name: Install Node.js dependencies
       run: npm ci

       - name: Build CSS with PostCSS
       run: npm run build:css

       - name: Setup Hugo
       uses: peaceiris/actions-hugo@v2
       with:
           hugo-version: "latest"
           extended: true

       - name: Build the site
       run: hugo --minify

       - name: Deploy to GitHub Pages
       uses: peaceiris/actions-gh-pages@v3
       if: github.ref == 'refs/heads/main'
       with:
           github_token: ${{ secrets.HUGO_DEPLOY }}
           publish_dir: ./public
   ```

10. Пушим репозиторий и деплоим сайт на `gh-pages`
