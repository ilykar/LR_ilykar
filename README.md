# ☕ Система управления кофейней

## 🏗️ Структура проекта

### Program.cs
```csharp
using System;
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("# СИСТЕМА УПРАВЛЕНИЯ КОФЕЙНЕЙ");
        Console.WriteLine("=" + new string('=', 50));

        var cashier = new Cashier("Елизавета", 1);
        var barista = new Barista("Илья", 2);

        var espresso = new Beverage("Эспрессо", 100m, Volume.S);
        var cappuccino = new Beverage("Капучино", 150m, Volume.M);
        var tiramisu = new Dessert("Тирамису", 200m, false);
        var glutenFreeCake = new Dessert("Морковный торт", 180m, true);

        // Сценарий 1: Простой заказ эспрессо
        Console.WriteLine("\n# СЦЕНАРИЙ 1: Простой заказ");
        Console.WriteLine("-" + new string('-', 30));

        var order1 = new Order();
        order1.AddItem(espresso);
        order1.DisplayOrder();

        cashier.DoWork(order1);
        barista.DoWork(order1);
        order1.UpdateStatus(OrderStatus.Completed);

        // Сценарий 2: Сложный заказ с добавками
        Console.WriteLine("\n# СЦЕНАРИЙ 2: Сложный заказ с добавками");
        Console.WriteLine("-" + new string('-', 30));

        var order2 = new Order();

        // Создаем капучино с молоком и сиропом
        var cappuccinoWithAddons = new SyrupAddon(new MilkAddon(cappuccino));
        order2.AddItem(cappuccinoWithAddons);

        // Десерт без добавок
        order2.AddItem(tiramisu);

        // Еще один эспрессо с корицей
        var espressoWithCinnamon = new CinnamonAddon(espresso);
        order2.AddItem(espressoWithCinnamon, 2);

        order2.DisplayOrder();

        cashier.DoWork(order2);
        barista.DoWork(order2);
        order2.UpdateStatus(OrderStatus.Completed);

        // Сценарий 3: Заказ с безглютеновым десертом
        Console.WriteLine("\n# СЦЕНАРИЙ 3: Заказ для клиента с аллергией");
        Console.WriteLine("-" + new string('-', 30));

        var order3 = new Order();
        order3.AddItem(glutenFreeCake);
        order3.AddItem(new MilkAddon(espresso));

        order3.DisplayOrder();

        cashier.DoWork(order3);
        barista.DoWork(order3);
        order3.UpdateStatus(OrderStatus.Completed);

        Console.WriteLine($"\n# Все заказы завершены! Рабочий день окончен.");
        Console.ReadKey();
    }
}
```

### Enum.cs
```csharp
public enum OrderStatus
{
    Created,
    Paid,
    InProgress,
    Ready,
    Completed
}

public enum Volume
{
    S = 250,
    M = 350,
    L = 450
}
```

### Products.cs
```csharp
public abstract class Product
{
    public string Name { get; protected set; }
    public decimal BasePrice { get; protected set; }

    protected Product(string name, decimal basePrice)
    {
        Name = name;
        BasePrice = basePrice;
    }

    public virtual string GetDescription()
    {
        return Name;
    }

    public virtual decimal GetCost()
    {
        return BasePrice;
    }
}

public class Beverage : Product
{
    public Volume Volume { get; private set; }

    public Beverage(string name, decimal basePrice, Volume volume)
        : base(name, basePrice)
    {
        Volume = volume;
    }

    public override string GetDescription()
    {
        return $"{base.GetDescription()} ({Volume} - {(int)Volume}мл.)";
    }
}

public class Dessert : Product
{
    public bool IsGlutenFree { get; private set; }

    public Dessert(string name, decimal basePrice, bool isGlutenFree)
        : base(name, basePrice)
    {
        IsGlutenFree = isGlutenFree;
    }

    public override string GetDescription()
    {
        string glutenInfo = IsGlutenFree ? " (Без глютена)" : " (Содержит глютен)";
        return base.GetDescription() + glutenInfo;
    }
}
```

### Addons.cs
```csharp
public abstract class Addon : Product
{
    protected Product Product { get; }

    protected Addon(Product product, string addonName, decimal addonPrice)
        : base(addonName, addonPrice)
    {
        Product = product;
    }
}

public class MilkAddon : Addon
{
    public MilkAddon(Product product)
        : base(product, "Молоко", 50m)
    {
    }

    public override string GetDescription()
    {
        return $"{Product.GetDescription()}, с молоком";
    }

    public override decimal GetCost()
    {
        return Product.GetCost() + BasePrice;
    }
}

public class SyrupAddon : Addon
{
    public SyrupAddon(Product product)
        : base(product, "Сироп", 30m)
    {
    }

    public override string GetDescription()
    {
        return $"{Product.GetDescription()}, с сиропом";
    }

    public override decimal GetCost()
    {
        return Product.GetCost() + BasePrice;
    }
}

public class CinnamonAddon : Addon
{
    public CinnamonAddon(Product product)
        : base(product, "Корица", 20m)
    {
    }

    public override string GetDescription()
    {
        return $"{Product.GetDescription()}, с корицей";
    }

    public override decimal GetCost()
    {
        return Product.GetCost() + BasePrice;
    }
}
```

### OrderItems.cs
```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class OrderItem
{
    public Product Product { get; }
    public int Quantity { get; }

    public OrderItem(Product product, int quantity)
    {
        Product = product;
        Quantity = quantity;
    }

    public decimal GetCost()
    {
        return Product.GetCost() * Quantity;
    }

    public string GetDescription()
    {
        return $"{Product.GetDescription()} x{Quantity} - {GetCost()} руб";
    }
}

public class Order
{
    private static int _nextOrderId = 1;

    public int OrderId { get; }
    public OrderStatus Status { get; private set; }
    private List<OrderItem> Items { get; }
    public decimal TotalPrice => CalculateTotal();

    public Order()
    {
        OrderId = _nextOrderId++;
        Status = OrderStatus.Created;
        Items = new List<OrderItem>();
    }

    public void AddItem(Product product, int quantity = 1)
    {
        var existingItem = Items.FirstOrDefault(item => item.Product.GetDescription() == product.GetDescription());

        if (existingItem != null)
        {
            Items.Add(new OrderItem(product, quantity));
        }
        else
        {
            Items.Add(new OrderItem(product, quantity));
        }

        Console.WriteLine($"# Добавлено: {product.GetDescription()} x{quantity}");
    }

    public void RemoveItem(OrderItem item)
    {
        Items.Remove(item);
    }

    public decimal CalculateTotal()
    {
        return Items.Sum(item => item.GetCost());
    }

    public void UpdateStatus(OrderStatus newStatus)
    {
        Console.WriteLine($"# Статус заказа #{OrderId} изменен: {Status} → {newStatus}");
        Status = newStatus;
    }

    public void DisplayOrder()
    {
        Console.WriteLine($"\n# ЗАКАЗ #{OrderId} (Статус: {Status})");
        Console.WriteLine("=" + new string('=', 40));

        foreach (var item in Items)
        {
            Console.WriteLine($"  {item.GetDescription()}");
        }

        Console.WriteLine($"\n  ИТОГО: {TotalPrice:C}");
        Console.WriteLine("=" + new string('=', 40));
    }
}
```

### Employees.cs
```csharp
using System.Threading;
using System;

public abstract class Employee
{
    public string Name { get; }
    public int EmployeeId { get; }

    protected Employee(string name, int employeeId)
    {
        Name = name;
        EmployeeId = employeeId;
    }

    public abstract void DoWork(Order order);
}

public class Cashier : Employee
{
    public Cashier(string name, int employeeId) : base(name, employeeId) { }

    public override void DoWork(Order order)
    {
        Console.WriteLine($"\n# КАССИР {Name} начинает обработку заказа #{order.OrderId}");
        AcceptPayment(order);
    }

    private void AcceptPayment(Order order)
    {
        Console.WriteLine($"  Принята оплата: {order.TotalPrice:C}");
        order.UpdateStatus(OrderStatus.Paid);
        Console.WriteLine($"  Кассир {Name} передал заказ #{order.OrderId} бариста");
    }
}

public class Barista : Employee
{
    public Barista(string name, int employeeId) : base(name, employeeId) { }

    public override void DoWork(Order order)
    {
        Console.WriteLine($"\n# БАРИСТА {Name} начинает готовить заказ #{order.OrderId}");
        PrepareOrder(order);
    }

    private void PrepareOrder(Order order)
    {
        order.UpdateStatus(OrderStatus.InProgress);

        Console.WriteLine("  Приготовление напитков...");
        Thread.Sleep(1000);
        Console.WriteLine("  Добавление добавок...");
        Thread.Sleep(500);

        order.UpdateStatus(OrderStatus.Ready);
        Console.WriteLine($"  Бариста {Name} завершил приготовление заказа #{order.OrderId}");
    }
}
```
