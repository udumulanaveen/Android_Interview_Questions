1. Difference between Shared and State Flow ?

Ans: StateFlow :StateFlow starts with a value you give it and shares it as soon as someone starts listening. 
It works mostly like LiveData. LiveData stops sharing automatically when the screen isn't showing anymore. With
StateFlow, you’ll have to stop sharing manually, but if you use repeatOnLifeCycle, you can achieve similar behaviour as
live data. 

import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

val counterStateFlow = MutableStateFlow(0) // Holds the initial state of the counter

fun main() = runBlocking {
    // Launch a coroutine to collect the StateFlow
    launch {
        counterStateFlow.collect { value ->
            println("Counter StateFlow: $value")
        }
    }

    // Simulate updating the counter state
    launch {
        repeat(5) { count ->
            delay(500)
            counterStateFlow.value = count + 1 // Update state with new counter value
        }
    }

    // Wait for some time before ending the program
    delay(3000)
}

StateFlow always holds the latest value (counterStateFlow.value), so new collectors get the current state immediately when they start collecting.
In this example, the state (counter value) is being updated every 500 milliseconds, and all collectors get the latest value.

SharedFlow: StateFlow only emits last known value , whereas in SharedFlow you can configure how many previous values to
be emitted. If you want emitting and collecting repeated values , got with sharedflow

import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

val notificationSharedFlow = MutableSharedFlow<String>() // Shared flow for notifications

fun main() = runBlocking {
    // Launch a coroutine to collect the SharedFlow (collector 1)
    launch {
        notificationSharedFlow.collect { notification ->
            println("Collector 1 received: $notification")
        }
    }

    // Emit notifications
    launch {
        delay(100) // Add slight delay before starting to emit
        notificationSharedFlow.emit("Notification 1")
        notificationSharedFlow.emit("Notification 2")
        delay(100)
        notificationSharedFlow.emit("Notification 3")
    }

    // Launch a second coroutine to collect SharedFlow (collector 2)
    launch {
        delay(250) // Start collecting after some delay
        notificationSharedFlow.collect { notification ->
            println("Collector 2 received: $notification")
        }
    }

    // Wait for some time before ending the program
    delay(1000)
}

SharedFlow broadcasts events to all current collectors.
In this example, Collector 2 starts collecting after the first two notifications have already been emitted, so it only receives "Notification 3".
Collector 1, however, receives all the notifications because it started collecting earlier.


Key Differences:

StateFlow:

Keeps the latest state and delivers it to new collectors.
Ideal for representing observable states like UI state in Android development.

SharedFlow:

Emits values to all collectors that are active at the time of emission.
It does not retain the latest value, so collectors may miss events if they weren't collecting when the value was emitted.
Suitable for broadcasting events like messages, notifications, or one-time actions.
Both are great tools for reactive programming in Kotlin, and you can choose between them based on whether you need to maintain state (StateFlow) or broadcast events (SharedFlow).










