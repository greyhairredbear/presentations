---
# try also 'default' to start simple
theme: default

# some information about your slides, markdown enabled
title: Introduction to property based testing with Kotest

# author field for exported PDF
author: Peter Laggner
# keywords field for exported PDF, comma-delimited.
keywords: property based testing, kotest, kotlin

# enable presenter mode, can be boolean, 'dev' or 'build'
presenter: true
# filename of the export file
exportFilename: intro-property-based-testing-kotest

colorSchema: light

# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki

# https://sli.dev/guide/drawing
drawings:
  persist: false
  
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left

# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true

fonts:
  # for code blocks, inline code, etc.
  mono: Jetbrains Mono
---

#### An introduction to 

# Property based testing 

#### with 

<img class="absolute left-100 w-40" src="/kotest-logo-with-text.png">

<!--
Shoutout to Karli for inviting me - I hope you won't regret it in the forseeable future

Disclaimer: This talk borrows quite a bit from Scott Wlaschin's talk 
"A lazy programmer's guide to writing thousands of tests"
-->

---

# Peter Laggner
 
<div grid="~ cols-[40px_1fr] gap-y4" items-center justify-center>
  <div>🏡</div><div><b>Based</b> in Graz</div>
  <div>🧑‍💻</div><div><b>Working</b> remotely at <a href="https://www.cargonexx.com">Cargonexx</a></div>
  <div>🚚</div><div><b>Modelling and solving</b> a vehicle routing problem<br>using Kotlin and Timefold</div> 
  <div i-devicon-kotlin text-xl/><div><b>Kotlin</b> enthusiast since 2017</div>
</div>

<div my-10 grid="~ cols-[40px_1fr] gap-y4" items-center justify-center>
  <div i-ri-github-line op50 text-xl/>
  <div><a href="https://github.com/greyhairredbear" target="_blank">greyhairredbear</a></div>
  <div i-ri-stack-overflow-line op50 text-xl/>
  <div><a href="https://stackoverflow.com/users/4929939/greyhairredbear" target="_blank">greyhairredbear</a></div>
</div>

<img class="w-40 border-black b-3 rounded-full abs-tr mt-32 mr-48" src="/profile-pic-self.jpeg">

--- 
layout: image-right
image: ./tour-planner-map.png
---

# Motivation

<v-clicks>

- Vehicle Routing Problem (VRP)
  - Complex domain
  - Computationally complex
- Build trust with potential users
  - Ensure complex problem is solved correctly

</v-clicks>

<!-- 

- At cargonexx, we aim to create a transport managment platform
  - Main persona: Dispatchers 

- Dispatcher's life:
  - context switches
  - phone calls every 5 minutes
  - unforeseen events
  - lots of adaption on the fly
  - **we want to ease that burden**
- Complex domain: 
  - delivery time windows 
  - return loads 
  - vehicle requirements
  - regulations on driving time
  - head knowledge of dispatchers 
    - several months of on-the-job training
- keep computationally complex in mind - we will come back to this later
-->

--- 

# A simple exercise*
<br>

<div text-center>
<i>"Write a function that adds two numbers..."</i>
</div>
<br>
<br>

<div v-click>

### ...using altered form of ping-pong TDD

- you're paired with a colleague 
- your job: Only write the tests

</div>
<div class="absolute text-sm right-30px bottom-30px">
*shamelessly adapted from Scott Wlaschin
</div>

---

# Meet your colleague

<v-clicks>

- Some say, he can be a bit tricky to work with
- Others say, he is a bit cynical about TDD
- Some even call him "Enterprise Developer from Hell" (EDFH)
- But how bad can it get?

</v-clicks>

--- 

# Let's get to it

````md magic-move
```kotlin
add(1, 2) shouldBe 3
```

```kotlin
fun add(a: Int, b: Int): Int = 3
```

```kotlin
add(1, 2) shouldBe 3

add(29, 13) shouldBe 42

add(-1, 5) shouldBe 4

add(1024, 1024) shouldBe 2048
```

```kotlin
fun add(a: Int, b: Int): Int = when {
    a == 1 && b == 2 -> 3
    a == 29 && b == 13 -> 42
    a == -1 && b == 5 -> 4
    a == 1024 && b == 1024 -> 2048
    else -> 31415926 // ¯\_(ツ)_/¯
}
```

```kotlin
// (╯°□°）╯︵ ┻━┻
```

````

--- 

# Let's try random input

<v-click>

```kotlin
import kotlin.random.Random

repeat(1000) {
    val a = Random.nextInt(Int.MIN_VALUE / 2, Int.MAX_VALUE / 2)
    val b = Random.nextInt(Int.MIN_VALUE / 2, Int.MAX_VALUE / 2)
    
    add(a, b) shouldBe a + b
}
```

</v-click>

<arrow v-click x1="359" y1="400" x2="359" y2="330" color="#ff2020" width="3" arrowSize="1" />
<div v-click>
<br>
EDFH doesn't really seem to bring out the best in us 
</div>

<!--
- Note that for the randoms here, we need to take care we don't overflow
- We want to take the high road here and not duplicate implementation
-->

---

# Property based testing - Commutativity

<v-click>

````md magic-move

```kotlin
import kotlin.random.Random

repeat(1000) {
    val a = Random.nextInt(Int.MIN_VALUE / 2, Int.MAX_VALUE / 2)
    val b = Random.nextInt(Int.MIN_VALUE / 2, Int.MAX_VALUE / 2)
    
    add(a, b) shouldBe add(b, a)
}
```

```kotlin
fun add(a: Int, b: Int): Int = a * b
```


````

</v-click>

---

# Property based testing - Successor

<v-click>

````md magic-move

```kotlin
import kotlin.random.Random

repeat(1000) {
    val a = Random.nextInt(Int.MIN_VALUE / 2, Int.MAX_VALUE / 2)
  
    add(add(a, 1), 1) shouldBe add(a, 2)
}
```

```kotlin
fun add(a: Int, b: Int): Int = 42
```

````

</v-click>

<!-- 
- adding 1 `n` times to `a` should be the n-th successor of `a`
- (we could generalize that test)
 -->

---

# Property based testing - Identity

<v-click>

````md magic-move

```kotlin
import kotlin.random.Random

repeat(1000) {
    val a = Random.nextInt(Int.MIN_VALUE / 2, Int.MAX_VALUE / 2)
  
    add(a, 0) shouldBe a
}
```

```kotlin
fun add(a: Int, b: Int): Int = a + b
```

````
</v-click>

<!-- 
- Admittedly, we might not have somebody that malicious sitting next to us...
- ... but edge cases still might be missed
- Reduce the chance of edge cases being missed
- Using random inputs in a real environment is more complicated than in the example
 -->

---

# Revisiting random inputs

How to ...

<v-clicks>

- ... make your test data generation reusable?
- ... reproduce test runs using random data?
- ... ensure seeds are deterministic for a test run?
- ... shrink your test data?
  - <i>Shrinking</i>: Get the least complex input that fails your test 

</v-clicks>

<!-- 
- Using random inputs in a real environment is more complicated than in the example
- While all of this is certainly no rocket science, it's probably not wise to reinvent the wheel
--> 

---

# Kotest's property test framework

<v-clicks>

- API for random input generation
- Lots of generators available (just like Kotest assertions)
- Test shrinking
- Nice documentation

</v-clicks>

<!-- 
Now Kotest's property test framework makes all this possible

Let's dive into how we can use Kotest for writing our property tests

and also, what sorts of property tests we can write
-->

---

# Real example - VRP input pre-processing

<v-clicks>

- Every load entity makes computation harder
- Grouping loads with same pickup and dropoff locations is beneficial (usually)

</v-clicks>

<v-click>

````md magic-move

```kotlin
class Load(
    val id: String,
    val measurements: Measurements,
    val requirements: List<String>,
    pickupTimeWindow: Interval,
    val pickupLocation: Location,
    dropoffTimeWindow: Interval,
    val dropoffLocation: Location,
    val planningDate: LocalDate,
    val containedLoadIds: List<String> = listOf(id)
) { ... }
```

```kotlin
fun List<Load>.grouped(
    loadGroupingConfig: LoadGroupingConfig
): List<Load>
```

```kotlin
data class LoadGroupingConfig(
    // loading (milli)meters, weight in kg
    val maxMeasurements: Measurements,
    val minTimeWindowOverlapInHours: Long,
)
```

````

</v-click>

---

# Real example - VRP input pre-processing

<v-clicks>

**Property**: Grouping a second time should not change the outcome

```kotlin 
checkAll(
    inputLoadGen, 
    groupingConfigGenerator
) { input, groupingConfig ->
    val result = input.grouped(groupingConfig)
    result shouldContainExactlyInAnyOrder result.grouped(groupingConfig)
}
```

</v-clicks>

<!-- Explain `checkAll` function -->

---

# Real example - VRP input pre-processing

<v-clicks>

**Property**: Don't drop any loads during grouping

```kotlin
checkAll(
    inputLoadGen, 
    groupingConfigGenerator
) { input, groupingConfig ->
    input.grouped(groupingConfig).flatMap {
        it.containedLoadIds
    } shouldContainExactlyInAnyOrder input.map { it.id }
}
```

</v-clicks>

---

# Real example - VRP input pre-processing

<v-clicks>

**Property**: Don't assign loads to more than one group

```kotlin
checkAll(
    inputLoadGen, 
    groupingConfigGenerator
) { input, groupingConfig ->
  val result = input.grouped(groupingConfig)
  result.flatMap { it.containedLoadIds }.shouldNotContainDuplicates()
}
```

</v-clicks>

---

# Real example - VRP input pre-processing

<v-clicks>

**Property**: Don't add more loads during grouping

```kotlin
checkAll(
    inputLoadGen, 
    groupingConfigGenerator
) { input, groupingConfig ->
  input.grouped(groupingConfig).count() shouldBeLessThanOrEqual 
      input.count()
}
```

</v-clicks>

---

# Real example - VRP input pre-processing

<v-clicks>

**Property**: Adhere to maximum measurements of grouping configuration for grouping loads

```kotlin
forAll(inputLoadGen, groupingConfigGenerator) { input, groupingConfig ->
    val groupedLoadsMeasurements =
        input
            .grouped(groupingConfig)
            .filter { it.containedLoadIds.count() > 1 }
            .map { it.measurements }
  
    groupedLoadsMeasurements.none {
        it.ldmm > groupingConfig.maxMeasurements.ldmm ||
            it.weightInKg > groupingConfig.maxMeasurements.weightInKg
    }
}
```

</v-clicks>

<!-- Note difference of checkAll and forAll -->


---

# Generators

<v-clicks>

DSL for generating random input

```kotlin
fun givenLoadGroupingConfigGenerator(): Arb<LoadGroupingConfig> = 
    arbitrary {
        LoadGroupingConfig(
          givenMeasurementGenerator().bind(),
          Arb.long(1L..96).bind(),
        )
    }
```

</v-clicks>

---

# Assumptions

<v-clicks>

Filter test data with assertions

```kotlin
checkAll(givenVehicleGenerator()) {
    val firstAction = it.actions.firstOrNull()
    assume {
        firstAction.shouldNotBeNull()
        firstAction.isOnSameVehicleAsRelatedAction.shouldBeTrue()
    }

    firstAction!!.isScheduledBeforeRelatedAction.shouldBeTrue()
}
```

</v-clicks>

---

# Remarks


<v-click>Property testing has limitations</v-click>

<v-clicks>

- Sometimes too coarse
- Finding properties is not always easy

</v-clicks>

<br>
<v-click>BUT</v-click>

<v-clicks>

- Helps you think about your program in a more general way
- Great for detection of edge cases
- Property tests are less prone to change

</v-clicks>

<br>
<v-click>
<b>Bottom line:</b> 
Mix property tests with your regular example based tests!
</v-click>

<!-- edge cases: found quite a few that hadn't been thought of before -->

---

# References

<v-clicks>

- Scott Wlaschin on property testing: https://fsharpforfunandprofit.com/series/property-based-testing/
- Kotest's property testing framework: <br> https://kotest.io/docs/proptest/property-based-testing.html
- Slides created with slidev: https://sli.dev
- This presentation: https://github.com/greyhairredbear/presentations <br> (`/intro-property-based-testing`)

</v-clicks>

<!-- 
If you want to learn more about property testing, absolutely check out scott wlaschin's website
-->

---
layout: center
---

# Questions

<!-- 
further interesting properties:
- invertible operation (example with minus based on addition)
- different paths, same destination (addition associativity)
- hard to prove, easy to verify
- test oracle
-->

<!--

-->
