| 方法                | 参数       |                                        |
| ------------------- | ---------- | -------------------------------------- |
| runAsync            | Runnable   | 执行一个没有返回值的任务               |
| supplyAsync         | Supplier   | 执行一个有返回值的任务                 |
| thenRunAsync        | Runnable   | 任务1执行完后执行任务2                 |
| thenAcceptAsync     | Consumer   | 任务1执行完后执行任务2                 |
| thenApplyAsync      | Function   | 任务1执行完后执行任务2                 |
| thenAcceptBothAsync | BiConsumer | 任务1和任务2执行完后,执行任务3         |
| thenCombineAsync    | BiFunction | 任务1和任务2执行完后,执行任务3         |
| runAfterBothAsync   | Runnable   | 任务1和任务2执行完后,执行任务3         |
| allOf               |            | 等待所有任务执行结束                   |
| anyOf               |            | 只要任意一个任务执行完成就可以         |
| acceptEitherAsync   | Consumer   | 两个任务中的其中一个执行完后,执行任务3 |
| applyToEitherAsync  | Function   | 两个任务中的其中一个执行完后,执行任务3 |
| runAfterEitherAsync | Runnable   | 两个任务中的其中一个执行完后,执行任务3 |

