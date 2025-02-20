# Anagami ERP – модул „Отчети“

Модулът „Отчети“ (`reports`) е самостоятелно изнесена част от **Anagami ERP** – мултитенант бизнес платформа върху Laravel 10 (PHP 8.3, Blade, Bootstrap 4, jQuery). Тук е събран целият код, който изгражда менюто „Отчети“: контролери, Yajra DataTables, Excel експорти, Blade изгледи, планираните имейл отчети (PDF), езиковите ключове и миграциите за правата за достъп.

> **Частно / вътрешно ползване.** Репозиторията е private и съдържа собствен код на Anagami. Не е самостоятелно инсталируем пакет – разчита на останалата част от приложението (виж „Зависимости“) и не трябва да се публикува или разпространява извън организацията.

Файловете запазват **оригиналните си относителни пътища** спрямо корена на Laravel проекта, така че съдържанието може да се копира директно върху приложението.

## Отчети в модула

Всички отчети са зад маршрути под префикс `account/` и права от модул `reports` (повечето изискват ниво `all`).

| Отчет | Маршрут (име) | Право | Какво прави (според кода) |
|-------|---------------|-------|---------------------------|
| **Отчет по задачи** | `task-report.index` | `view_task_report` | DataTable със задачи (код, проект, клиент, изпълнител, краен срок, статус) с филтри и графика (`task-report.chart`). |
| **Задачи по служител** | `employee-wise-task-report` | `view_task_report` | По служител: общо назначени, завършени, чакащи и просрочени задачи. |
| **Консолидиран отчет по задачи** | `consolidated-task-report` | `view_task_report` | По задача: оценени часове и общо отработени часове. |
| **Отчет по време (time log)** | `time-log-report.index` | `view_time_log_report` | Записи на време: проект, задача, служител, начало/край, часове/минути, бележка, клиент, приходи; графика (`time-log-report.chart`) и общо време (`time-log-report.time`). |
| **Консолидиран отчет по време** | `time-log-consolidated.report` | `view_time_log_report` | Обобщение по служител/проект/задача/дата: общо часове, минути, почивки, приходи. |
| **Отчет по време по проекти** | `project-wise-timelog.report` | `view_time_log_report` | По служител и проект: общо часове и продължителност на почивките; експорт в Excel (`project-wise-timelog.export`, `ProjectwiseTimeLogExport`). |
| **Седмични таймшийтове** | `time-log-weekly-report.index` | `view_time_log_report` | Седмици по служител със статус на одобрение (одобрен / чакащ / чернова) и пряк ръководител. |
| **Чакащи седмични таймшийтове** | `weekly-pending-time-log-report.report` | `view_time_log_report` | Служители и седмици с неподадено/чакащо време. |
| **Финансов отчет** | `finance-report.index` | `view_finance_report` | Плащания по проект/фактура/сума/дата/статус; филтри по период, проект, клиент, валута; 4 графики (тенденция, по проект, сравнение по месеци, топ клиенти – Chart.js) през `finance-report.chart`. |
| **Приходи срещу разходи** | `income-expense-report.index` | `view_income_expense_report` | Графика на завършените плащания (приход) спрямо разходите за избран период, с общи суми в валутата на компанията. |
| **Отчет за разходите** | `expense-report.index` | `view_expense_report` | Разходи (артикул, цена, служител, доставчик, банкова сметка, дата, бележка/документ, статус) + графики по период и по категория (`expense-report.chart`). |
| **Разходи по категория** | `expense-report.expense_category_report` | `view_expense_report` | Обща сума на разходите по категория за избран период. |
| **Отчет за отпуски** | `leave-report.index` / `.show` | `view_leave_report` (all/added/owned/both) | По служител: одобрени и чакащи отпуски; детайл по тип отпуск за избран период. |
| **Отчет за квоти на отпуски** | `leave-report.leave_quota` | `view_leave_report` | По служител: общо отпуск и оставащ; детайл по година/месец (`leave-report.employee-leave-quota`); експорт `LeaveQuotaReportExport`. |
| **Отчет за присъствие** | `attendance-report.index` | `view_attendance_report` | По служител: присъствия, отсъствия, допълнителни дни, половин дни, отработени часове, календарна матрица на месеца (вкл. почивни дни). |
| **Отчет за сделки / лийдове** | `lead-report.index` | `view_lead_report` | По агент: общо, спечелени и загубени сделки, обща и конвертирана стойност; профил на агент (`lead-report.profile`). |
| **Контакти (лийдове)** | `lead-report.lead_contact`, `lead-report.total_contact` | `view_lead` | Списък на лийд контактите със статус, собственик, адрес, потребителски полета. |
| **Прогноза / среден размер на сделка** | `lead-report.chart` | `view_lead_report` | По година, pipeline и категория: сделки по етап и месец (Frappe Charts), експорт в Excel (`deal-report.export`, `DealReportExport`: брой затворени сделки, обща и средна стойност, спечелени). |
| **Отчет за продажби** | `sales-report.index` | `view_sales_report` | Фактури: дата на плащане, №, клиент, стойност, платено, данъчна основа, отстъпка, банкова сметка. |

### Планирани имейл отчети (PDF)

| Отчет | Команда / график | Какво прави |
|-------|------------------|-------------|
| Дневен отчет по време | `send-daily-timelog-report`, ежедневно в 02:30 | За всяка активна компания изпраща PDF (`timelog-report.blade.php`) по имейл на избраните роли (`log_time_for.daily_report_roles`). |
| Месечен отчет за присъствие | `send-monthly-attendance-report`, ежемесечно | За компании с включена настройка `monthly_report` изпраща PDF (`attendance-report.blade.php`) на избраните роли (`monthly_report_roles`). |

## Структура на папките

| Папка | Файлове | Съдържание |
|-------|--------:|------------|
| `app/Http/Controllers/` | 10 | `Attendance`, `Expense`, `Finance`, `IncomeVsExpense`, `Lead`, `Leave`, `Sales`, `Task`, `Timelog` `ReportController` + `TimelogWeeklyApprovalController` |
| `app/DataTables/` | 17 | Yajra DataTables на всички отчети (вкл. `EmployeeWiseTaskDataTable`, `LeadContactDataTable`, `consolidatedTaskReportDataTable`) |
| `app/Exports/` | 3 | `DealReportExport`, `LeaveQuotaReportExport`, `ProjectwiseTimeLogExport` (maatwebsite/excel) |
| `app/Console/Commands/` | 2 | `SendDailyTimelogReport`, `SendMonthlyAttendanceReport` |
| `app/Events/`, `app/Listeners/`, `app/Notifications/` | 2 + 2 + 2 | `DailyTimeLogReport*`, `MonthlyAttendance*` (имейл отчетите) |
| `resources/views/reports/` | 30 | Изгледи по отчет: `attendance`, `expense`, `finance`, `income-expense`, `lead`, `leave`, `leave-quota`, `sales`, `tasks`, `timelogs` |
| `resources/views/` (корен) | 2 | `timelog-report.blade.php`, `attendance-report.blade.php` – PDF шаблони |
| `resources/views/mail/` | 2 | Markdown имейл шаблони (`mail/timelog/`, `mail/attendance/`) |
| `resources/lang/eng/` | 1 | `reports-keys.php` – референтен списък на използваните езикови ключове |
| `database/migrations/` | 3 | Права за отчети за лийдове, продажби и отпуски |
| `routes/` | 1 | `reports.routes.php` – извадка от `routes/web.php` (референция) |
| Корен | 2 | `README.md`, `.gitignore` |
| **Общо** | **79** | |

Разпределение на изгледите в `resources/views/reports/`: `attendance` 1, `expense` 4, `finance` 2, `income-expense` 2, `lead` 5, `leave` 3, `leave-quota` 2, `sales` 1, `tasks` 4, `timelogs` 6.

### Езикови файлове

В основното приложение няма отделен `reports*.php`; текстовете на отчетите са в споделените `resources/lang/eng/{app,modules,messages,email,placeholders}.php` (напр. `app.menu.taskReport`, `modules.reports.*`, `modules.incomeVsExpenseReport.*`). Затова тук е добавен само `resources/lang/eng/reports-keys.php` – генериран чрез статичен анализ на `__()`, `@lang()` и `trans()` във файловете на модула (202 разрешени ключа, подредени по оригиналния файл). Файлът **не се зарежда автоматично** и не замества оригиналните; служи за проверка какви ключове трябва да съществуват (и за превод). Динамично сглобени ключове и имената на правата (`permissions.php`) не са обхванати.

### Миграции

Включени са само миграциите, които са изцяло или основно за отчети:

- `2023_02_17_052112_add_permissions_for_lead_report.php`
- `2023_03_24_081626_add_permission_for_sales_report.php`
- `2023_11_21_112750_add_permissions_for_leave_reports.php` – освен правата за отчета за отпуски съдържа и две несвързани промени (`project_time_logs.memo` → `text` и изтриване на настройката `clock-in-notification`).

## Зависимости от останалата система (НЕ са включени)

Модулът е разрез от монолитното приложение и разчита на следното, което трябва да съществува в целевия проект:

- **Multi-tenancy:** trait `HasCompany` и глобалният скоуп `CompanyScope` (автоматично филтриране по `company_id`), `ActiveScope`; контролерите наследяват `AccountBaseController`.
- **Модели:** `User`, `Company`, `Role`, `EmployeeDetails`, `Invoice`, `Payment`, `Tax`, `Currency`, `Expense`, `ExpensesCategory`, `Task`, `TaskboardColumn`, `TaskCategory`, `TaskLabelList`, `Project`, `ProjectTimeLog`, `Attendance`, `Holiday`, `Leave`, `LeaveType`, `Lead`, `LeadAgent`, `LeadStatus`, `LeadSource`, `LeadCategory`, `LeadPipeline`, `PipelineStage`, `Deal`, `LogTimeFor`, `CustomField`, `CustomFieldGroup`.
- **Помощни класове и функции:** `App\Helper\Reply`, `App\Helper\Common` и глобалните функции от `app/Helper/start.php` – напр. `user()`, `company()`, `abort_403()`, `canDataTableExport()`, `companyToDateString()`, `currency_format()`, `user_modules()`, `attendance_setting()`.
- **Права и менюта:** модулът `reports` и правата `view_*_report` са дефинирани в `app/Models/Module.php`, `ModuleSetting.php`; менюто „Отчети“ е в `resources/views/sections/menu.blade.php`. Нищо от това не е копирано.
- **Споделени изгледи и компоненти:** `layouts.app`, `sections.datatable_css`, `sections.datatable_js`, `sections.daterange_css`, `sections.daterange_js`, Blade компоненти (`x-filters.*`, `x-forms.*`, `x-alert` и др.), както и `employees.leaves_quota`, включван от `reports/leave-quota/show.blade.php`.
- **Регистрации:** `App\Console\Kernel` (команди и график), `EventServiceProvider` (`DailyTimeLogReportEvent`, `MonthlyAttendanceEvent` → listeners).
- **Споделени контролери:** `LeavesQuotaController::exportAllLeaveQuota` (маршрут `leave_quota.export_all_leave_quota`) и `WeeklyTimesheetController` остават в основното приложение.
- **Таблици:** `payments`, `invoices`, `expenses`, `project_time_logs`, `tasks`, `attendances`, `leaves`, `deals`, `lead_agents`, `log_time_for`, `attendance_settings` и др.

### Съзнателно изключено

- `Modules/Payroll/.../PayrollReportController` и експортите на заплати – принадлежат на плъгин модула Payroll.
- Миграция `2023_02_13_045833_add_report_column_in_log_time_table.php` – смесена с несвързани промени (чат, SMTP, съобщения).
- Регистрациите на права/менюта в `Module.php`, `menu.blade.php`, `Kernel.php`, `EventServiceProvider.php`, `routes/web.php` (споделени файлове; за маршрутите е дадена референция).
- `vendor/`, `node_modules/`, `public/` ресурси (Frappe Charts, Chart.js), `.env`, `CLAUDE.md`.

## Изисквания

- PHP ≥ 8.2 (приложението работи на 8.3), Laravel 10
- `yajra/laravel-datatables` (Yajra DataTables)
- `maatwebsite/excel` (експорти в `.xlsx`)
- `barryvdh/laravel-dompdf` (PDF в имейл отчетите)
- Фронтенд: jQuery, Bootstrap 4, daterangepicker, **Frappe Charts** (`public/vendor/jquery/frappe-charts.min.iife.js`, `public/vendor/graph/frappechart.js`) и **Chart.js** (`public/vendor/jquery/Chart.min.js`; на места – CDN `cdn.jsdelivr.net/npm/chart.js`)

## Интеграция

1. Копирайте съдържанието на `app/`, `resources/` и `database/` върху корена на Laravel проекта (пътищата съвпадат).
2. Пренесете маршрутите от `routes/reports.routes.php` в групата `['middleware' => ['auth', 'multi-company-select', 'email_verified'], 'prefix' => 'account']` на `routes/web.php`.
3. Уверете се, че са регистрирани: командите в `App\Console\Kernel` (`send-daily-timelog-report` – всеки ден в 02:30, `send-monthly-attendance-report` – ежемесечно) и събитията `DailyTimeLogReportEvent`/`MonthlyAttendanceEvent` в `EventServiceProvider`.
4. Изпълнете миграциите: `php artisan migrate`.
5. Проверете, че езиковите ключове от `resources/lang/eng/reports-keys.php` съществуват в езиковите файлове на приложението (за `bg` и други езици се превеждат отделно).
6. Изчистете кеша: `php artisan view:clear && php artisan route:clear && php artisan config:clear`.

## Бележка

Кодът е вътрешна собственост на Anagami и е предназначен за ползване само в рамките на организацията `anagamibg`.
