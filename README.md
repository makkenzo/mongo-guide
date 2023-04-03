# Гайд на подключение, обновление и создание базы данных MongoDB Cloud .NET Framework Windows Forms

1. Зарегистрируйтесь на https://www.mongodb.com/ и создайте кластер. 

2. Введите имя пользователя и пароль в соответствующие поля (можно сгенерировать пароль кнопкой). Сохраните логин и пароль.

3. Перейдите во вкладку "Security" и удалите все адреса, которые уже есть в разделе "Network Access". Затем нажмите кнопку "ADD IP ADDRESS", выберите "ALLOW ACCESS FROM ANYWHERE" и подтвердите.

4. Перейдите во вкладку "Database" (если не перенаправлены автоматически). Нажмите кнопку "Browse Collections", затем "Add my own data". Введите название базы данных и название коллекции (коллекцию можно удалить позже).

5. Нажмите кнопку "Connect" на странице кластера. Выберите "Connect your application" и "Driver: C# / .NET, Version: 2.13 or later". Скопируйте сгенерированный код.

6. В Visual Studio нажмите ПКМ по проекту и выберите "Управление пакетами NuGet…". Найдите пакет "MongoDB.Driver", выберите свой проект и установите пакет, соглашаясь со всем, что будет всплывать.

## a. В коде, где нужен доступ к базе данных, вставьте скопированный код из Connect. В начале файла добавьте следующие using:

```csharp
using MongoDB.Driver;
using MongoDB.Bson;
```

1. в ссылке что у вас будет скопирована после Connect
(пример(не копировать, у каждого разная ссылка))

```csharp
var settings = MongoClientSettings.FromConnectionString("mongodb+srv://<username>:<password>@dexcluster.mx0indr.mongodb.net/?retryWrites=true&w=majority");
```

2. меняем <username> и <password> на логин и пароль пользователя что вы создали когда создавали кластер (если хотите поменять или забыли пароль то переходим в Security => Database Access, вы увидите пользователя что вы создали, его можно отредактировать либо создать новый). у вас будет переменная var database - в ней хранится ваша база данных, не забудьте в конце в кавычках поменять на название своей бд.


## b. Создайте новый класс и напишите следующий код:

```csharp
public class название_класса
{
    private static readonly Lazy<IMongoDatabase> lazyDatabase = new Lazy<IMongoDatabase>(() =>
    {
        // Вставляем код скопированный с Connect
        return client.GetDatabase("название бд");
    });

    public static IMongoDatabase GetDatabase()
    {
        return lazyDatabase.Value;
    }
}
```

1. В любом файле, где вам нужен объект базы данных, получите его следующим образом:

```csharp
var database = YourClassName.GetDatabase("db_name");
```

## Использование

- Чтобы получить доступ к коллекции(таблице):
```csharp
var collection = database.GetCollection<BsonDocument>("collection_name");
```

- Чтобы найти документ(запись) в коллекцию(таблицу):
```csharp
var filter = Builders<BsonDocument>.Filter.Eq("уникальное поле", "значение уникального поля");
```
"filter" служит для фильтрации коллекции по фильтру, в примере выше фильтруются документы по уникальному полю. Например, в моей коллекции в каждой записи лишь один username, поэтому я могу отфильтровать все записи по username и результатом будет лишь один документ.

- После создания фильтра нужно осуществить поиск по этому фильтру:
```csharp
var result = collection.Find(filter).FirstOrDefault();
```

В переменную result записывается результат поиска по нашему фильтру и уже из этой переменной мы можем брать данные, например:
```csharp
result.GetValue("значение поля которое мы хотим вытянуть").AsString;
```

AsString - в каком типе вы вытягиваем значение, если в коллекции у этого поля тип string, то вытягиваем AsString, если Int32, то вытягиваем как AsInt32 и т.д.
Пример:
```csharp
var filter = Builders<BsonDocument>.Filter.Eq("username", "vladik");
credentials.FName = result.GetValue("password").AsString;
```

- Чтобы добавить документ в коллекцию:
```csharp
var filter = Builders<BsonDocument>.Filter.Eq("уникальное поле","уникальное значение");
var result = collection.Find(filter).FirstOrDefault();
var document = new BsonDocument
{
    { "поле", "значение"},
    { "поле", "значение"},
    { "поле", "значение"}
};
сollection.InsertOne(document);
```

- Чтобы обновить документ в коллекции:
Ищем документ по фильтру и создаем переменную в которой будут обновленные данные:
```csharp
var update = Builders<BsonDocument>.Update.Set("поле которое мы хотим обновить", "значения поля которое мы хотим обновить");
```

Если мы хотим обновить несколько полей, то после .Set(...) не ставим ;, а продолжаем добавляя новый .Set(...).Set(...);
```csharp
var update = Builders<BsonDocument>.Update
                        .Set("поле", "значение")
                        .Set("поле", "значение");
```
