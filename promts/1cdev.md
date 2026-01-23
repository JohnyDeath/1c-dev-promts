---
inclusion: always
---

## 1C:Enterprise 8 Development Guidelines

### Platform & Language

**1C:Enterprise 8** (1С:Предприятие 8) with **BSL** (Built-in Scripting Language)

- Write ALL code, comments, and identifiers in Russian
- Use UTF-8 encoding exclusively (never Windows-1251/CP1251)
- Follow metadata-driven architecture with managed forms (управляемые формы)

### Code Style Rules

**Naming:** PascalCase for all identifiers (procedures, functions, variables, parameters)

**Structure:**
```bsl
Процедура ВыполнитьОперацию(Параметр1, Параметр2) Экспорт
    Перем ЛокальнаяПеременная;
    Если Условие Тогда
        // Комментарий только для сложной бизнес-логики
    КонецЕсли;
КонецПроцедуры

Функция ПолучитьСумму(Документ) Экспорт
    Возврат Результат;
КонецФункции
```

**Mandatory rules:**
- Add `Экспорт` keyword to all public procedures/functions
- Keep code compact - no empty lines inside procedure/function bodies
- Minimize comments - only for complex business logic, not obvious operations
- Never use try-catch (`Попытка...Исключение...КонецПопытки`) for database reads

### Execution Context Directives

**ALWAYS specify one of these directives before each procedure/function:**

- `&НаКлиенте` - Client-side execution (UI interactions, user input)
- `&НаСервере` - Server-side with form context access
- `&НаСервереБезКонтекста` - Server-side stateless (preferred for client→server calls)

**Pattern for client-server interaction:**
```bsl
&НаКлиенте
Процедура КнопкаНажатие(Команда)
    Результат = ВыполнитьНаСервере();
КонецПроцедуры

&НаСервереБезКонтекста
Функция ВыполнитьНаСервере()
    // Серверная логика с доступом к БД
    Возврат Результат;
КонецФункции
```

### Project Architecture

**Directory structure:**
- `src/cf/` - Main configuration metadata
  - `Documents/` - Business documents (счета, накладные, акты)
  - `Catalogs/` - Reference books (справочники)
  - `DataProcessors/` - Batch operations (обработки)
  - `Reports/` - Report generators (отчеты)
  - `CommonModules/` - Shared logic (общие модули)
  - `InformationRegisters/` - Data storage (регистры сведений)
  - `AccumulationRegisters/` - Balances/turnovers (регистры накопления)
- `src/epf/` - External DataProcessors source (внешние обработки)
- `src/erf/` - External Reports source (внешние отчеты)
- `src/cfe/` - Configuration extensions source
- `utils/` - Compiled binaries

**External reports/processors structure (native 1C XML format, NOT EDT/MDO):**
```
ИмяОбработки/
├── ИмяОбработки.xml              # Root metadata with InternalInfo
└── ИмяОбработки/                 # Nested folder (same name)
    ├── Ext/
    │   └── ObjectModule.bsl      # Object module (optional)
    └── Forms/
        ├── ИмяФормы.xml          # Form metadata
        └── ИмяФормы/
            └── Ext/
                ├── Form.xml      # Form layout and attributes
                └── Form/
                    └── Module.bsl # Form module code
```

**Critical XML requirements:**
- Root XML MUST contain `<InternalInfo>` with `<xr:ContainedObject>` and `<xr:GeneratedType>`
- Use correct ClassId in `<xr:ClassId>`:
  - External Report (erf): `e41aff26-25cf-4bb6-b6c1-3f478a75f374`
  - External DataProcessor (epf): `c3831ec8-d8d5-4f93-8a22-f9bfae07327f`
- Reference `ExampleCode1c/` for correct XML structure

### Query Language (1C Query Language)

**Standard query pattern:**
```bsl
Запрос = Новый Запрос;
Запрос.Текст = 
"ВЫБРАТЬ
|    Документ.Ссылка КАК Ссылка,
|    Документ.Дата КАК Дата,
|    Документ.Номер КАК Номер
|ИЗ
|    Документ.ИмяДокумента КАК Документ
|ГДЕ
|    Документ.Проведен = ИСТИНА
|    И Документ.ПометкаУдаления = ЛОЖЬ";

Результат = Запрос.Выполнить();
Выборка = Результат.Выбрать();
Пока Выборка.Следующий() Цикл
    // Обработка строки
КонецЦикла;
```

**Query rules:**
- Use pipe `|` for line continuation in query text
- Always alias tables with `КАК`
- Filter out deleted items: `ПометкаУдаления = ЛОЖЬ`
- For documents, check posting status: `Проведен = ИСТИНА`

### Critical Development Rules

1. **API Validation** - ALWAYS use MCP tools before writing code with platform APIs
2. **Context Directives** - Every procedure/function MUST have `&НаКлиенте`, `&НаСервере`, or `&НаСервереБезКонтекста`
3. **Transactions** - Wrap all data modifications:
   ```bsl
   НачатьТранзакцию();
   Попытка
       // Изменения данных
       ЗафиксироватьТранзакцию();
   Исключение
       ОтменитьТранзакцию();
       ВызватьИсключение;
   КонецПопытки;
   ```
4. **Error Handling** - Use `Попытка...Исключение...КонецПопытки` for data modifications, NOT for reads
5. **Performance** - Never execute queries inside loops; use batch operations with temporary tables
6. **Document Posting** - Documents must be posted (`Проведен = Истина`) to generate register records
7. **References vs Objects** - Use `Ссылка` type for references, not full object instances
8. **User Messages** - Never use `Сообщить()`. Use common modules:
   - Client: `ОбщегоНазначенияКлиент.СообщитьПользователю()`
   - Server: `ОбщегоНазначения.СообщитьПользователю()`
9. **Encoding** - All files MUST be UTF-8, never use Windows-1251/CP1251
10. **Language** - All code, comments, identifiers MUST be in Russian
