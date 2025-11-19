Sure thing! Let’s walk through **delegate examples in C#**—clean, clear, and with enough punch that your students won’t fall asleep mid-semicolon.

---

# ✅ **1. Basic Delegate Declaration & Usage**

A delegate is basically a type-safe function pointer. It holds references to methods with a matching signature.

### **Example:**

```csharp
// Declare delegate
public delegate void Notify(string message);

public class Notifier
{
    public void SendNotification(string message)
    {
        Console.WriteLine($"Notification: {message}");
    }
}

class Program
{
    static void Main()
    {
        var notifier = new Notifier();
        
        // Assign method to delegate
        Notify notifyDelegate = notifier.SendNotification;

        // Invoke delegate
        notifyDelegate("Backup completed successfully.");
    }
}
```

---

# ✅ **2. Delegate with Multiple Subscribers (Multicast Delegate)**

Delegates can call multiple methods — like a WhatsApp broadcast list but without the family drama.

### **Example:**

```csharp
public delegate void Logger(string msg);

class Program
{
    static void LogToConsole(string msg) => Console.WriteLine("Console: " + msg);
    static void LogToFile(string msg) => Console.WriteLine("File: " + msg); // Pretend file

    static void Main()
    {
        Logger logger = LogToConsole;
        logger += LogToFile;  // Add another subscriber

        logger("System started.");
    }
}
```

---

# ✅ **3. Delegate as a Parameter (Higher-Order Functions)**

You pass a delegate into a method to customize behavior.

### **Example:**

```csharp
public delegate int Calculation(int a, int b);

class Program
{
    static int Execute(int a, int b, Calculation operation)
        => operation(a, b);

    static int Add(int a, int b) => a + b;
    static int Multiply(int a, int b) => a * b;

    static void Main()
    {
        Console.WriteLine(Execute(5, 3, Add));
        Console.WriteLine(Execute(5, 3, Multiply));
    }
}
```

---

# ✅ **4. Using Built-in Delegates: Action, Func, Predicate**

### **Action (no return):**

```csharp
Action<string> printer = msg => Console.WriteLine(msg);
printer("Hello from Action!");
```

### **Func (returns value):**

```csharp
Func<int, int, int> sum = (a, b) => a + b;
Console.WriteLine(sum(10, 20));
```

### **Predicate (returns bool):**

```csharp
Predicate<int> isEven = x => x % 2 == 0;
Console.WriteLine(isEven(10)); // true
```

---

# ✅ **5. Callback Delegate**

Useful when you want to notify when an operation is done.

```csharp
public delegate void Callback();

class Program
{
    static void DownloadFile(Callback done)
    {
        Console.WriteLine("Downloading...");
        done(); // notify when done
    }

    static void OnDownloadCompleted() =>
        Console.WriteLine("Download completed!");

    static void Main()
    {
        DownloadFile(OnDownloadCompleted);
    }
}
```

---

# ✅ **6. Event with Delegate (classic usage)**

Events are basically “delegates with manners.”

```csharp
public delegate void MessageHandler(string message);

public class Chat
{
    public event MessageHandler? OnMessage;

    public void Send(string message)
    {
        OnMessage?.Invoke(message);
    }
}

class Program
{
    static void Main()
    {
        var chat = new Chat();

        chat.OnMessage += msg => Console.WriteLine("Received: " + msg);

        chat.Send("Hello from event!");
    }
}
```