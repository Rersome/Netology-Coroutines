# Домашнее задание к занятию «10. Coroutines: Scopes, Cancellation, Supervision»

Выполненное задание прикрепите ссылкой на ваши GitHub-проекты в личном кабинете студента на сайте [netology.ru](https://netology.ru).

## Вопросы: Cancellation

### Вопрос №1

Отработает ли в этом коде строка `<--`? Поясните, почему да или нет.

### Ответ: Нет, строка не отработает.
### Объяснение: job.cancelAndJoin() вызывается через 100 мс, что происходит до истечения 500 мс, необходимых для выполнения корутин

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
    }
    delay(100)
    job.cancelAndJoin()
}
```

### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

### Ответ: Нет, строка не отработает.
### Объяснение: Корутин child запускается и ожидает 500 мс, однако после 100 мс child.cancel() вызывается, что приведет к отмене выполнения корутины

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        val child = launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
        delay(100)
        child.cancel()
    }
    delay(100)
    job.join()
}
```

## Вопросы: Exception Handling

### Вопрос №1

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

### Ответ: Нет, строка не отработает.
### Объяснение: Исключение запускается в корутине, и поскольку launch возвращает Job, а не Deferred, оно не перехватывает исключение родительским блоком try-catch

```kotlin
fun main() {
    with(CoroutineScope(EmptyCoroutineContext)) {
        try {
            launch {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

### Ответ: Да, строка отработает.
### Объяснение: Тут используется coroutineScope, который перехватывает исключения. Исключение, выбрасываемое из coroutineScope, будет поймано родительским блоком try-catch

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

### Вопрос №3

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

### Ответ: Нет, строка не отработает.
### Объяснение: supervisorScope работает иначе: исключения в дочерних корутинах не передаются родителю. Следовательно, исключение не обрабатывается в родительском блоке try-catch

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

### Вопрос №4

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

### Ответ: Нет, строка не отработает.
### Объяснение: Поскольку coroutineScope создает новый контекст, исключения, выбрасываемые из дочерних корутин, не будут пойманы родителем. Первое исключение, выбрасываемое из второй корутины, приведет к отмене выполнения

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    Thread.sleep(1000)
}
```

### Вопрос №5

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

### Ответ: Да, строка отработает.
### Объяснение: В этом сценарии используется supervisorScope, и поскольку дочерние корутины работают независимо, исключение из одной из них не отменит выполнение остальных. Исключение, выбрасываемое из первой корутины, будет перехвачено в блоке try-catch

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

### Вопрос №6

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

### Ответ: Да, строка отработает.
### Объяснение: Хотя исключение выбрасывается, оно происходит в корутине, которая вызывается после линии с println("ok"). Этот вызов выполняется, и оба корутины "завершат" свою работу, прежде чем произойдет выброс исключения

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```

### Вопрос №7

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

### Ответ: Да, строка отработает.
### Объяснение: Использование SupervisorJob() позволяет дочерним корутинам работать независимо, так что исключение, выбрасываемое в одном из дочерних корутин, не повлияет на выполнение других

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```
