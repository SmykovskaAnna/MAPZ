# Лабораторна робота №3
### Побудова діаграм об'єктів, компонентів та розгортання

**Студент:** Смиковська Анна Володимирівна, ЗІПЗ-23-1

**Предметна область:** Проєктування та розробка інтерактивного вебзастосунку для формування, моніторингу та аналітики персональних звичок.

**Назва системи:** HabitTracker

## 1. Мета роботи

Ознайомитися з діаграмами об'єктів, компонентів і розгортання UML, побудувати їх для системи HabitTracker та узгодити з діаграмою класів лабораторної роботи №2.

## 2. Діаграма об'єктів

Для діграми об'єктів обрано п'ять класів із діаграми класів: `ЗареєстрованийКористувач`, `Категорія`, `Звичка`, `ЗаписВиконання` і `Нагадування`. Їхні екземпляри та значення атрибутів демонструють звичку користувача, її категорію, виконання та нагадування.

Лінки відповідають зв'язкам класової діаграми: користувач - звичка, категорія - звичка, звичка - запис виконання та звичка - нагадування. На діаграмі об'єктів вони показані звичайними лінками між екземплярами; кратності й ромби з діаграми класів не дублюються. Для звички показано одне налаштоване нагадування; це відповідає множинності `1 - 0..*` у діаграмі класів.

### 2.1. PlantUML-код діаграми

Файл діаграми: `lab03_object_diagram.puml`.

```plantuml
@startuml
title Діаграма об'єктів HabitTracker
skinparam objectAttributeIconSize 0
skinparam linetype ortho

object "anna : ЗареєстрованийКористувач" as anna {
  ідентифікатор = 1
  ім'я = "Анна"
  електроннаПошта = "anna@example.com"
  датаРеєстрації = "2026-09-01"
  активний = true
  часовийПояс = "Europe/Warsaw"
}
object "health : Категорія" as health {
  ідентифікатор = 1
  назва = "Здоров'я"
  опис = "Фізичне та ментальне здоров'я"
}
object "morningExercise : Звичка" as habit {
  ідентифікатор = 101
  назва = "Ранкова зарядка"
  опис = "15 хвилин фізичних вправ"
  періодичність = "Щодня"
  статус = "Активна"
}
object "completion22Nov : ЗаписВиконання" as record {
  ідентифікатор = 1001
  дата = "2026-11-22"
  виконано = true
  коментар = "Виконано о 08:10"
}
object "morningReminder : Нагадування" as reminder {
  ідентифікатор = 201
  час = "08:00"
  активне = true
  дніТижня = "Пн-Нд"
}

anna -- habit : створює
health -- habit : категорія
habit -- record : запис виконання
habit -- reminder : має
@enduml
```

## 3. Діаграма компонентів

Систему HabitTracker розкладено на шість внутрішніх логічних компонентів-модулів за бізнес-можливостями та зовнішній сервіс сповіщень. Ці модулі входять до одного застосунку на Java та Spring Boot і не розгортаються як окремі мікросервіси. Зв’язки інтерфейсів показані сполучниками ball-and-socket.

| **Компонент** | **Наданий інтерфейс** | **Споживає** |
| --- | --- | --- |
| Керування користувачами | `IUserService` | - |
| Керування звичками | `IHabitService` | `IUserService`, `ICategoryService` |
| Керування категоріями | `ICategoryService` | - |
| Відстеження прогресу | `IProgressService` | `IHabitService` |
| Сервіс нагадувань | `IReminderService` | `IHabitService`, `IUserService`, `INotificationAPI` |
| Аналітика | `IAnalyticsService` | `IProgressService`, `IUserService` |
| Зовнішній сервіс сповіщень | `INotificationAPI` | — |

### 3.1. PlantUML-код діаграми

Файл діаграми: `lab03_component_diagram.puml`.

```plantuml
@startuml
title Діаграма компонентів HabitTracker
skinparam componentStyle uml2
left to right direction

component "Керування користувачами" as User
component "Керування звичками" as Habit
component "Керування категоріями" as Category
component "Відстеження прогресу" as Progress
component "Сервіс нагадувань" as Reminder
component "Аналітика" as Analytics
component "Зовнішній сервіс сповіщень" as Notification

User -0)- Habit : IUserService
Category -0)- Habit : ICategoryService
Habit -0)- Progress : IHabitService
Habit -0)- Reminder : IHabitService
User -0)- Reminder : IUserService
Progress -0)- Analytics : IProgressService
User -0)- Analytics : IUserService
Notification -0)- Reminder : INotificationAPI

interface "IReminderService" as IReminder
interface "IAnalyticsService" as IAnalytics
Reminder - IReminder
Analytics - IAnalytics
@enduml
```

## 4. Діаграма розгортання

Діаграма визначає п'ять вузлів інфраструктури: пристрій клієнта, вебсервер, сервер застосунків, сервер бази даних і зовнішній сервіс сповіщень. Шість логічних компонентів із попередньої діаграми входять до одного артефакту `habittracker.jar` на основі Java та Spring Boot. Thymeleaf формує HTML-сторінки, а Bootstrap використовується для стилів і клієнтських компонентів. Дані зберігаються в MySQL. Nginx приймає HTTPS-запити та передає їх серверу застосунків.

| **Вузол** | **Розміщені артефакти** |
| --- | --- |
| Пристрій клієнта | Веббраузер |
| Вебсервер | Nginx, Bootstrap (CSS, JavaScript) |
| Сервер застосунків | `habittracker.jar` (Java, Spring Boot, Thymeleaf) |
| Сервер бази даних | `habittracker_db` (MySQL) |
| Зовнішній сервіс сповіщень | Notification API |

Шляхи комунікації підписано протоколами: HTTPS між клієнтом і вебсервером, HTTP між Nginx і Spring Boot (зворотне проксіювання), MySQL/TCP через порт 3306 між застосунком і базою даних, HTTPS/REST API між застосунком і зовнішнім сервісом сповіщень.

### 4.1. PlantUML-код діаграми

Файл діаграми: `lab03_deployment_diagram.puml`.

```plantuml
@startuml
title Діаграма розгортання HabitTracker

node "Пристрій клієнта" as Client {
  artifact "Веббраузер" as Browser
}
node "Вебсервер" as Web {
  artifact "Nginx" as Nginx
  artifact "Bootstrap CSS / JavaScript" as Bootstrap
}
node "Сервер застосунків" as App {
  artifact "habittracker.jar\n(Java, Spring Boot, Thymeleaf)" as Application
}
node "Сервер бази даних" as DB {
  database "habittracker_db\n(MySQL)" as Database
}
node "Зовнішній сервіс сповіщень" as Notification {
  artifact "Notification API" as NotificationAPI
}

Client -- Web : HTTPS
Web -- App : HTTP (reverse proxy)
App -- DB : MySQL / TCP 3306
App -- Notification : HTTPS / REST API
@enduml
```

## 5. Висновки

Під час виконання лабораторної роботи 3 побудовано діаграми об'єктів, компонентів і розгортання системи HabitTracker. Діаграма об'єктів показує конкретний приклад стану системи. Діаграма компонентів пояснює логічну структуру застосунку та взаємодію його модулів. Діаграма розгортання відображає обраний стек Java, Spring Boot, Thymeleaf, Bootstrap і MySQL, розміщення артефактів і протоколи обміну даними.

Діаграма компонентів найкраще пояснює логічну будову системи сторонньому розробнику: вона показує межі відповідальності модулів і контракти між ними. Діаграми об'єктів і розгортання доповнюють її прикладом стану системи та інфраструктурою виконання.