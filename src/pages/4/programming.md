# Программирование UI

Теперь, когда имеются виджеты, которые должны взаимодействовать друг с другом,
вы можете подумать, что данная часть является небольшой, но сложной в реализации.
Если вы так думаете, то вы полностью ошибаетесь, потому что данная часть будет
самой легкой частью программы для реализации. Начнем с функции `main`, которую
мы возьмем из первой главы в качестве шаблона.

```rust
fn main() {
    // Инициализация GTK перед началом работы.
    if gtk::init().is_err() {
        eprintln!("failed to initialize GTK Application");
        process::exit(1);
    }

    // Инициализация начального состояния пользовательского интерфейса.
    let app = App::new();

    // Напишите код работы ваших виджетов здесь.

    // Сделать видимыми все виджеты пользовательского интерфейса
    app.window.show_all();

   // Запустим главный цикл событий (_event loop_) GTK.
    gtk::main();
}
```

Мы запрограммируем кнопку **Post** так, чтобы она принимала элементы **title**
и **tags**, также как и буфер текстовой панели **content**. Далее мы пропустим
строки из этих виджетов через HTML-макроопределение _horrorshow_ и напишем
получившийся результат в текстовый буфер **right_pane**. Код для программирования
кнопки выглядит так:

```rust
{
    // Программирование кнопки **Post** на принятие входных значений из
    // левой панели, произведение необходимого обновление HTML-кода на
    // правой панели. Подготовка к увеличение значения счетчиков...
    let title = app.content.title.clone();
    let tags = app.content.tags.clone();
    let content = app.content.content.clone();
    let right_pane = app.content.right_pane.clone();
    app.header.post.connect_clicked(move |_| {
        let inputs = (title.get_text(), tags.get_text(), get_buffer(&content));
        if let (Some(title), Some(tags), Some(content)) = inputs {
            right_pane.set_text(&generate_html(&title, &tags, &content));
        }
    });
}
```

Заметьте, получить текста из элемента очень просто. Для этого нужно всего лишь
вызвать метод **get_text()**, который возвращает **Option<String>**. Получение
текста из текстового буфера немного сложнее, поэтому вам необходимо использовать
функцию, которая была рекомендована в начале этой главы. Эта функция написана так:
```rust
/// Получить содержимое текстового буфера в виде строки.
fn get_buffer(buffer: &TextBuffer) -> Option<String> {
    let start = buffer.get_start_iter();
    let end = buffer.get_end_iter();
    buffer.get_text(&start, &end, true)
}
```

Вы также заметите интересный шаблон проектирования (паттерн) в Rust, которая
сильно упростила нам проверку наличия всех входных данных при получение входных
данных, все это произошло перед тем как что-то сделали с входными значениями.
Синтаксис **if let** в Rust работает не только с шаблонами (паттернами),
но и с кортежами (_tuple_), так что вы можете проверять несколько входных
данных в кортеже одновременно, так же, как вы бы делали это в **match**.
```rust
let inputs = (title.get_text(), tags.get_text(), get_buffer(&content));
if let (Some(title), Some(tags), Some(content)) = inputs {
    right_pane.set_text(&generate_html(&title, &tags, &content));
}
```
Нам еще предстоит определить функцию **generate_html**, и это будет
завершающей частью реализации приложения. Самым простым способом
использования макроопределения **html!** является его подстановка в качестве
аргумента в макроопределение **format!**. Наша функция будет выглядеть так,
хотя вы вольны реализовать HTML-макроопределение по своему усмотрению.
```rust
/// Генерирует HTML, который будет показан на правой панели.
fn generate_html(title: &str, tags: &str, content: &str) -> String {
    format!{
        "{}",
        html!{
            article {
                header {
                    h1 { : &title }
                    div(class="tags") {
                        @ for tag in tags.split(':') {
                            div(class="tag") { : tag }
                        }
                    }
                }
                @ for line in content.lines().filter(|x| !x.is_empty()) {
                    p { : line }
                }
            }
        }
    }
}
```
Синтаксис приведенного выше кода должен быть довольно читаемым. Мы создаем
пару тэгов **article**, которая содержит в себе пару тэгов **header** и
параграф **p** для каждой непустой линии из входных данных, полученных из
текстового буфера **content**. Внутри тэгов **header** есть заголовок **h1**,
который использует текст из поля для ввода названия как свой текст. Также там
есть элемент **div**, который содержит список тэгов, разделенных двоеточиями.
![head_pic](https://mmstick.github.io/gtkrs-tutorials/images/ch03_complete.png)

Имея все это на своих местах, у вас должна получиться работающая программа,
выглядящая как на изображении. 