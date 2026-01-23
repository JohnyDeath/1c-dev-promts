# 1cdev.md

## 1C:Enterprise 8 Development Guidelines

### Language & Platform

**1C:Enterprise 8** (1С:Предприятие 8) using **BSL** (Built-in Scripting Language)

- All code, comments, identifiers in **Russian**
- Metadata-driven architecture
- UTF-8 encoding only (never Windows-1251/CP1251)
- Managed forms (управляемые формы)

### Code Style

**Naming:** PascalCase for all identifiers

```bsl
Процедура ВыполнитьОперацию(Параметр1, Параметр2) Экспорт
    Перем ЛокальнаяПеременная;
    Если Условие Тогда
        // Код только для сложной логики
    КонецЕсли;
КонецПроцедуры

Функция ПолучитьСумму(Документ) Экспорт
    Возврат Результат;
КонецФункции
```

**Key rules:**
- Use `Экспорт` for public procedures/functions
- Minimal comments - only for complex business logic
- Compact code - no empty lines inside procedures/functions
- No unnecessary comments

### Execution Contexts

**Always specify context directive:**

- `&НаКлиенте` - Client-side UI logic
- `&НаСервере` - Server with form state access
- `&НаСервереБезКонтекста` - Stateless server (preferred for client→server calls)

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

### Architecture

**Project structure:**
- `src/cf/` - Configuration metadata
  - `Documents/` - Business documents
  - `Catalogs/` - Reference books
  - `DataProcessors/` - Batch operations
  - `Reports/` - Report generators
  - `CommonModules/` - Shared logic
  - `InformationRegisters/` - Data storage
  - `AccumulationRegisters/` - Balances/turnovers
- `utils/` - Compiled external processors (binary)
- `src/epf/`, `src/erf/`, `src/cfe/` - Utility source code

**Form structure:**
- `Forms/ФормаДокумента/Ext/Form.xml` - Layout
- `Forms/ФормаДокумента/Ext/Form/Module.bsl` - Logic

### Query Language

```bsl
Запрос = Новый Запрос;
Запрос.Текст = 
"ВЫБРАТЬ
|    Документ.Ссылка,
|    Документ.Дата
|ИЗ
|    Документ.ИмяДокумента КАК Документ
|ГДЕ
|    Документ.Проведен";

Результат = Запрос.Выполнить();
```

### Critical Rules

1. **Validate first** - Use MCP tools before code changes
2. **Transactions** - Wrap data modifications: `НачатьТранзакцию()` / `ЗафиксироватьТранзакцию()`
3. **Error handling** - Always use: `Попытка...Исключение...КонецПопытки`
4. **Performance** - Never query in loops, use batch operations
5. **Document posting** - Documents must be posted (`Проведен = Истина`) to create register records
6. **References** - Use `Ссылка` type, not object instances
7. **Context directives** - Always specify `&НаКлиенте`, `&НаСервере`, or `&НаСервереБезКонтекста`

