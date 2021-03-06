# Creating an ActiveRecord class

Castle ActiveRecord acts on what is called ActiveRecord types. Those are classes that use the ActiveRecordAttribute. They can be also extended from one of the ActiveRecord base classes, although this is not strictly necessary. There is a generic base class and another one that supports [Validators](validators.md).

## The ActiveRecordAttribute

The `ActiveRecordAttribute` is used to define a class as an *ActiveRecord type* and to associate mapping information.

The example below uses the parameterless constructor of the attribute:

using Castle.ActiveRecord;

```csharp
[ActiveRecord]
public class Product : ActiveRecordBase<Product>
{
    ...
```

For the situation above the programmer did not made explicit the table name nor the database schema. ActiveRecord then **assumes** the class `Product` is being mapped to a database table with **the same name**. The schema will be `null`.

The following code snippet make the table name explicit:

```csharp
using Castle.ActiveRecord;

[ActiveRecord("Products")]
public class Product : ActiveRecordBase<Product>
{
    ...
```

You can also use the Table and Schema properties to make the information more clear to newcomers:

```csharp
using Castle.ActiveRecord;

[ActiveRecord(Table="Products",Schema="dbo")]
public class Product : ActiveRecordBase<Product>
{
    ...
```

Please refer to the Reference Manual's Attributes article for further information.

## The ActiveRecordBase class

:information_source: An *ActiveRecord type* must have a parameterless contructor.

The use of a base class is optional for using ActiveRecord. However, using one of base classes is a simple way to expose data operations at the entity class.

The default base class is `ActiveRecordBase<T>``. This class is generic by purpose. It defines multiple methods for fetching objects of the type from the database that use the generic class as return type. This allows the use of those methods without explicitly casting the results down to the required class. Example:

```csharp
using Castle.ActiveRecord;

[ActiveRecord]
public class Product : ActiveRecordBase<Product>
{
    // mapping omitted
}

public class ClientCode
{
    public void UsesFindMethod(int id)
    {
        Product p = Product.Find(id);
        // use p
    }
}
```

If it is not possible to use a generic baseclass for some reason, the non-generic version of `ActiveRecordBase` can be used. The relevant find methods of this class are marked protected internal and must be overwritten to be used by client code:

```csharp
using Castle.ActiveRecord;

[ActiveRecord]
public class Product : ActiveRecordBase
{
    // mapping omitted

    public static Product Find(int id)
    {
        return (Product)FindByPrimaryKey(typeof(Product), id);
    }
}
```

The `ActiveRecordBase` class also exposes instance members to `Save`, `Create`, `Update` and `Delete`. The `Save` operation is able to find out if a class needs to be created or updated. `Create` and `Update` do not perform this check. More information on these methods can be found under ActiveRecord operations

:warning: **Warning:** If a class has an assigned primary key type (i.e. the primary key is not generated by the database) it is necessary to specify a value that marks unsaved instances or it is **not possible** to use `Save`. For those classes `Create` or `Update` are available only. See the section about primary key mappings for more information on this topic.