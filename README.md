**阅读目录**

* [步骤 1: 创建 TaskExCum 类](#_label0)
* [步骤 2: 实现 Run 方法](#_label1)
* [步骤 3: 实现 WhenAll 方法](#_label2)
* [步骤 4: 实现 WhenAllCore 方法](#_label3)
* [步骤 5: 添加异常处理逻辑](#_label4)
* [示例 1: 使用 Run 方法](#_label5)
* [示例 2: 使用 WhenAll 方法](#_label6)

# 实现 .NET 4\.0 下的 Task 类相似功能：TaskExCum 组件详解


## 引言


随着 .NET 技术的发展，异步编程模型逐渐成为现代应用程序开发中的标准实践之一。.NET 4\.5 引入了 `Task` 类，极大地简化了异步编程的过程。然而，许多遗留系统仍在使用 .NET 4\.0 或更低版本，这些版本并未直接支持 `Task` 类的全部功能。为此，我们开发了 `TaskExCum` 组件，旨在为 .NET 4\.0 提供与 .NET 4\.5 相似的 `Task` 功能，包括 `Task.Run()` 和 `Task.WhenAll()` 方法。


## 组件概述


`TaskExCum` 是一个静态类，提供了以下主要功能：


* **Run** 方法：用于异步执行任务，并获取任务的结果。
* **WhenAll** 方法：用于等待多个任务完成，并收集所有任务的结果。


## 实现步骤


接下来，我们将详细讲解 `TaskExCum` 组件的实现步骤，以便读者能够更好地理解其工作原理，并将其应用于自己的项目中。


[回到顶部](#_labelTop)### 步骤 1: 创建 `TaskExCum` 类


首先，我们需要创建一个静态类 `TaskExCum`，并在其中定义静态方法。



```
public static class TaskExCum
{
    // 方法定义将在后续步骤中添加
}

```

[回到顶部](#_labelTop):[milou加速器](https://xinminxuehui.org)### 步骤 2: 实现 `Run` 方法


`Run` 方法允许开发者异步执行任务，并获取任务的结果。我们为 `Run` 方法提供了两种重载形式，分别用于执行无返回值的操作（`Action`）和有返回值的操作（`Func`）。



```
public static Task<TResult> Run<TResult>(Func function)
{
#if NET45
    // 如果目标框架是 .NET 4.5 或更高版本，使用 Task.Run 方法
    return Task.Run(function);
#else
    // 如果目标框架是 .NET 4.0，使用 Task.Factory.StartNew 方法
    return Task.Factory.StartNew(
        function, 
        CancellationToken.None, 
        TaskCreationOptions.None, 
        TaskScheduler.Default
    );
#endif
}

public static Task Run(Action action)
{
#if NET45
    // 如果目标框架是 .NET 4.5 或更高版本，使用 Task.Run 方法
    return Task.Run(action);
#else
    // 如果目标框架是 .NET 4.0，使用 Task.Factory.StartNew 方法
    return Task.Factory.StartNew(
        action, 
        CancellationToken.None, 
        TaskCreationOptions.None, 
        TaskScheduler.Default
    );
#endif
}

```

**详细解释**：


* **条件编译**：使用 `#if NET45` 编译符号，当项目目标框架为 .NET 4\.5 及更高版本时，使用 `Task.Run` 方法。
* **Task.Factory.StartNew**：当项目目标框架为 .NET 4\.0 时，使用 `Task.Factory.StartNew` 方法来启动任务。
	+ **CancellationToken.None**：表示没有取消令牌，任务不会被外部取消。
	+ **TaskCreationOptions.None**：表示没有特殊的任务创建选项。
	+ **TaskScheduler.Default**：这是关键点之一。`TaskScheduler.Default` 表示使用默认的线程池调度器，这意味着任务会在线程池中的一个线程上执行，而不是每次都启动一个新的线程。这有助于提高性能和资源利用率。


[回到顶部](#_labelTop)### 步骤 3: 实现 `WhenAll` 方法


`WhenAll` 方法用于等待多个任务完成，并收集所有任务的结果。我们为 `WhenAll` 方法提供了多种重载形式，以支持不同类型的任务集合。



```
public static Task<TResult[]> WhenAll<TResult>(IEnumerable> tasks)
{
#if NET45
    // 如果目标框架是 .NET 4.5 或更高版本，使用 Task.WhenAll 方法
    return Task.WhenAll(tasks);
#else
    // 如果目标框架是 .NET 4.0，调用 WhenAllCore 方法
    return WhenAllCore(tasks);
#endif
}

public static Task<TResult[]> WhenAll<TResult>(params Task[] tasks)
{
#if NET45
    // 如果目标框架是 .NET 4.5 或更高版本，使用 Task.WhenAll 方法
    return Task.WhenAll(tasks);
#else
    // 如果目标框架是 .NET 4.0，调用 WhenAllCore 方法
    return WhenAllCore(tasks);
#endif
}

public static Task WhenAll(IEnumerable tasks)
{
#if NET45
    // 如果目标框架是 .NET 4.5 或更高版本，使用 Task.WhenAll 方法
    return Task.WhenAll(tasks);
#else
    // 如果目标框架是 .NET 4.0，调用 WhenAllCore 方法
    return WhenAllCore(tasks);
#endif
}

```

**详细解释**：


* **条件编译**：使用 `#if NET45` 编译符号，当项目目标框架为 .NET 4\.5 及更高版本时，使用 `Task.WhenAll` 方法。
* **WhenAllCore**：当项目目标框架为 .NET 4\.0 时，调用 `WhenAllCore` 方法来实现相同的功能。


[回到顶部](#_labelTop)### 步骤 4: 实现 `WhenAllCore` 方法


`WhenAllCore` 方法是 `WhenAll` 方法的核心实现，负责处理任务集合，等待所有任务完成，并收集结果或异常信息。



```
private static Task WhenAllCore(IEnumerable tasks)
{
    return WhenAllCore(tasks, (completedTasks, tcs) => tcs.TrySetResult(null));
}

private static Task<TResult[]> WhenAllCore<TResult>(IEnumerable> tasks)
{
    return WhenAllCore(tasks.Cast(), (completedTasks, tcs) =>
    {
        tcs.TrySetResult(completedTasks.Select(t => ((Task)t).Result).ToArray());
    });
}

private static Task<TResult> WhenAllCore<TResult>(IEnumerable tasks, Action> setResultAction)
{
    if (tasks == null)
    {
        throw new ArgumentNullException("tasks");
    }

    Contract.EndContractBlock();
    Contract.Assert(setResultAction != null);

    var tcs = new TaskCompletionSource();
    var array = (tasks as Task[]) ?? tasks.ToArray();

    if (array.Length == 0)
    {
        // 如果任务集合为空，直接设置结果
        setResultAction(array, tcs);
    }
    else
    {
        Task.Factory.ContinueWhenAll(array, completedTasks =>
        {
            var exceptions = new List();
            bool hasCanceled = false;

            foreach (var task in completedTasks)
            {
                if (task.IsFaulted)
                {
                    // 收集所有失败任务的异常信息
                    exceptions.AddRange(task.Exception.InnerExceptions);
                }
                else if (task.IsCanceled)
                {
                    // 检查是否有任务被取消
                    hasCanceled = true;
                }
            }

            if (exceptions.Count > 0)
            {
                // 如果有异常，设置异常结果
                tcs.TrySetException(exceptions);
            }
            else if (hasCanceled)
            {
                // 如果有任务被取消，设置取消结果
                tcs.TrySetCanceled();
            }
            else
            {
                // 如果没有异常且没有任务被取消，设置成功结果
                setResultAction(completedTasks, tcs);
            }
        }, CancellationToken.None, TaskContinuationOptions.ExecuteSynchronously, TaskScheduler.Default);
    }

    return tcs.Task;
}

```

**详细解释**：


* **参数验证**：检查传入的任务集合是否为 `null`，如果是，则抛出 `ArgumentNullException`。
* **TaskCompletionSource**：创建一个 `TaskCompletionSource` 对象，用于管理任务的完成状态。
* **任务转换**：将任务集合转换为数组，以便于后续处理。
* **任务数量检查**：如果任务集合为空，直接调用 `setResultAction` 设置结果。
* **等待所有任务完成**：使用 `Task.Factory.ContinueWhenAll` 方法等待所有任务完成。
	+ **异常处理**：遍历已完成的任务，收集所有失败任务的异常信息。
	+ **取消处理**：检查是否有任务被取消。
	+ **设置结果**：如果没有异常且没有任务被取消，调用 `setResultAction` 设置结果。
	+ **TaskScheduler.Default**：这里再次使用 `TaskScheduler.Default`，确保任务在默认的线程池中执行，而不是每次都启动新的线程。


[回到顶部](#_labelTop)### 步骤 5: 添加异常处理逻辑


为了确保组件的健壮性，我们还需要在 `WhenAllCore` 方法中添加异常处理逻辑，确保所有异常都能被捕获并正确处理。



```
private static void AddPotentiallyUnwrappedExceptions(ref List targetList, Exception exception)
{
    var ex = exception as AggregateException;
    Contract.Assert(exception != null);
    Contract.Assert(ex == null || ex.InnerExceptions.Count > 0);

    if (targetList == null)
    {
        targetList = new List();
    }

    if (ex != null)
    {
        // 如果异常是 AggregateException，添加其内部异常
        targetList.Add(ex.InnerExceptions.Count == 1 ? ex.InnerExceptions[0] : ex);
    }
    else
    {
        // 否则，直接添加异常
        targetList.Add(exception);
    }
}

```

**详细解释**：


* **异常类型检查**：检查传入的异常是否为 `AggregateException`。
* **异常列表初始化**：如果 `targetList` 为 `null`，则初始化一个新的列表。
* **异常添加**：根据异常的类型，将异常或其内部异常添加到列表中。


## 示例代码


为了帮助读者更好地理解如何使用 `TaskExCum` 组件，下面是一些示例代码。


[回到顶部](#_labelTop)### 示例 1: 使用 `Run` 方法



```
using System;
using System.Threading.Tasks;

class Program
{
    static void Main(string[] args)
    {
        try
        {
            // 异步执行任务并等待结果
            string result = TaskExCum.Run(() => "Hello from Task!").Result;
            Console.WriteLine(result);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }
    }
}

```

[回到顶部](#_labelTop)### 示例 2: 使用 `WhenAll` 方法



```
using System;
using System.Linq;
using System.Threading.Tasks;

class Program
{
    static void Main(string[] args)
    {
        try
        {
            // 创建多个任务
            var tasks = Enumerable.Range(1, 5).Select(i => TaskExCum.Run(() => i * i)).ToArray();
            
            // 等待所有任务完成并获取结果
            int[] results = TaskExCum.WhenAll(tasks).Result;
            
            // 输出结果
            foreach (var result in results)
            {
                Console.WriteLine(result);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }
    }
}

```

## 结论


通过 `TaskExCum` 组件，即使是在 .NET 4\.0 这样的老框架版本中，我们也能够享受到现代异步编程模型带来的便利。希望这个组件能够帮助那些需要在旧版 .NET 框架中实现异步操作的开发者们，提高他们的开发效率和代码质量。如果你有任何建议或改进意见，欢迎留言交流！




---


详情请看：[https://github.com/Bob\-luo/p/18515670](https://github.com)
以上就是关于 `TaskExCum` 组件的详细介绍。希望通过这篇文章，读者能够更好地理解和使用这个组件，从而在自己的项目中实现高效的异步编程。如果有任何问题或需要进一步的帮助，请随时留言！


