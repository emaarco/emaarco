# Simplifying Process Automation with bpmn-to-code: From BPMN Models to Process APIs

![A developer feeds a BPMN diagram card into a friendly generator machine that outputs a neat stack of code cards, turning models into a process API](./hero.jpg)

As a developer, I’ve always been driven by the desire to create clean and intelligent solutions. Whether the challenge is technical, architectural, or organizational, I aim for solutions that reduce friction  —  not just for me, but for others as well. One area in which I have repeatedly encountered small but annoying hurdles is the one I deal with most often: Process automation with BPMN and engines such as Zeebe. In this domain, in my role as a consultant, I help clients automate processes and enable them to work with BPMN and process engines effectively - no matter whether its about modeling or implementation.

One recurring issue I’ve seen  in there — both for myself and for those I support  —  is the need to manually copy technical configurations from BPMN models into the codebase. This includes things like element IDs, message names and worker types. It’s a tedious task that introduces opportunities for errors and increases cognitive load, especially as models evolve and configurations must be manually kept in sync with the code.

To address these issues, I’ve built a Maven and Gradle plugin called [**bpmn-to-code**](https://github.com/Miragon/bpmn-to-code). It generates what I like to call **Process APIs** — Kotlin or Java code artifacts created directly from your BPMN models that contain all the technical configurations needed to interact with a process engine.

Overall it pursues the goal to reduce manual effort, prevent errors, and ultimately foster clean and solutions for process automation. And maybe, just maybe, make developing with BPMN and process engines feel more easy and familiar to developers everywhere .

### 🕵️ Challenges When Using a Process Engine

Let’s be honest—working with process engines isn’t always as smooth as we’d like. Imagine you’ve been asked to automate a newsletter subscription process. Naturally, you’d begin by designing the process model in BPMN: first, a user submits a form, then receives a confirmation email, confirms their subscription, and finally gets a welcome email. Once the basic flow is defined, you can further refine the process to address exceptions and additional scenarios. Thus, in a more detailed form, the process model might look roughly like this:

![BPMN model of the newsletter subscription: after the form is submitted, a subprocess sends a confirmation mail and waits for confirmation, resending daily; after confirmation a welcome mail is sent, while a 3-day timeout aborts the registration and an invalid mail ends it as not possible](https://cdn-images-1.medium.com/max/800/1*e7uuMpzPFtFWtfRE5zXC1Q.png)

Once you have the process visually modeled, you move on to assigning technical configurations — such as element IDs, message names, task topics — and then deploy the model to your BPMN engine of choice, like Camunda or Zeebe. So far, so good. But then comes the coding part.

To make the process executable, you will implement service task workers, send messages to the engine, and write tests to validate the flow. At this stage, every piece of your code must reference the technical details from the BPMN model. To retrieve these details, you typically end up copying them manually from the model into your code. A nostalgic practice that, let’s be honest, shouldn’t still be a thing in 2025. As a result, you might write service-task workers that rely on hardcoded strings for worker types, as illustrated below.

```kotlin
@Component
class AbortRegistrationWorker(
    private val useCase: AbortSubscriptionUseCase
) : DefaultJobWorker(
    type = "newsletter.abortRegistration",
) {

    private val log = KotlinLogging.logger {}

    override fun executeTask(
        client: JobClient,
        job: ActivatedJob
    ): Map<String, Any?> {
        val input = job.getVariablesAsType(Input::class.java)
        log.debug { "Received job to abort registration: $input" }
        useCase.abort(SubscriptionId(input.subscriptionId))
        return emptyMap()
    }

    data class Input(val subscriptionId: UUID)
}
```

And It doesn’t stop there. If you’re writing tests as well, you could end up with multiple instances of these hardcoded strings — each one a potential source of trouble.

```kotlin
@Test
fun `happy path - user subscribes to newsletter`() {

    val subscriptionId = UUID.fromString("4a607799-804b-43d1-8aa2-bdcc4dfd9b86")

    processPort.submitForm(SubscriptionId(subscriptionId))
    waitForProcessInstanceHasReachedElement("Activity_ConfirmRegistration")

    processPort.confirmSubscription(SubscriptionId(subscriptionId))
    waitForProcessInstanceHasPassedElement("EndEvent_RegistrationCompleted")

    assertThatProcess().isCompleted()
    assertThatProcess().hasPassedElementsInOrder(
        "StartEvent_RequestReceived",
        "Activity_SendConfirmationMail",
        "Activity_ConfirmRegistration",
        "Activity_SendWelcomeMail",
        "EndEvent_RegistrationCompleted"
    )

    verify { sendConfirmationMailUseCase.sendConfirmationMail(SubscriptionId(subscriptionId)) }
    verify { sendWelcomeMailUseCase.sendWelcomeMail(SubscriptionId(subscriptionId)) }
    verify { abortSubscriptionUseCase wasNot Called }
    confirmVerified(sendConfirmationMailUseCase, sendWelcomeMailUseCase, abortSubscriptionUseCase)
}
```

To summarize: Not only is this approach frustrating, but it’s also a potential minefield. One typo a missed update can cause your code to diverge from your model. As the model evolves — what it inevitably does — you have to manually track down every reference in the code and update it. The potential for error is high, and the process is far from efficient.

Now, imagine you’re new to BPMN and process engines. You’ve barely heard of element IDs or service-task topics, and when you try to build something, you quickly run into these issues. Instead of seeing the potential, you feel frustrated. It’s easy to reject the technology altogether and miss out on the value it could bring.

Having spent years developing process-automation solutions and enabling clients to work with BPMN and process engines, I’ve seen these challenges firsthand. That’s why I decided to implement **bpmn-to-code**.

### 🚀 An Idea for a Solution

bpmn-to-code is a plugin available for both [Gradle](https://plugins.gradle.org/plugin/io.miragon.bpmn-to-code-gradle) and [Maven](https://central.sonatype.com/artifact/io.miragon/bpmn-to-code-maven). You can think of it like Swagger Codegen, but for BPMN & process automation with the corresponding engines.

Just as Swagger generates client code from OpenAPI definitions, **bpmn-to-code** generates a lightweight Process API from your BPMN models. This API file is a code-based representation of your process model — in the form of a Java or Kotlin file — that contains all the relevant technical configurations, such as element IDs, message IDs, worker types, and the process ID.

Currently, the plugin supports both **Camunda** 7 and **Zeebe**, and it’s built with extensibility in mind. This means additional engine support can be added as needed, making the tool flexible and future-ready.

### 🎮 The Plugin in Action

Let’s look at what actually changes when you use **bpmn-to-code** in your project. First things first — how do you get it running? If you’re using Gradle, just add the plugin declaration to your `build.gradle.kts`. If you are using Maven, please refer to the respective README on GitHub.

```kotlin
plugins {
    id("io.github.emaarco.bpmn-to-code-gradle") version "0.0.1-alpha"
}
```

Next, configure the generation task. In this task, you define where your BPMN models live, where the generated code should be stored, which language the API should be generated in, and which process engine you’re working with.

```kotlin
tasks.named("generateBpmnModelApi", GenerateBpmnModelsTask::class) {
    baseDir = projectDir.toString()
    filePattern = "src/main/resources/**/*.bpmn"
    outputFolderPath = "$projectDir/src/main/kotlin"
    packagePath = "de.emaarco.example"
    outputLanguage = OutputLanguage.KOTLIN
    processEngine = ProcessEngine.ZEEBE
}
```

Once you run this task, the plugin will generate your Process API. Returning to our newsletter subscription example, the plugin would generate a **Kotlin** or **Java** file that looks like this:

```kotlin
package com.example.process

@Suppress("unused")
object NewsletterSubscriptionProcessApiV1 {
    val PROCESS_ID: String = "newsletterSubscription"

    object Elements {
        val Timer_EveryDay: String = "Timer_EveryDay"
        val Timer_After3Days: String = "Timer_After3Days"
        val ErrorEvent_InvalidMail: String = "ErrorEvent_InvalidMail"
        val Activity_ConfirmRegistration: String = "Activity_ConfirmRegistration"
        val SubProcess_Confirmation: String = "SubProcess_Confirmation"
        val EndEvent_RegistrationAborted: String = "EndEvent_RegistrationAborted"
        val EndEvent_SubscriptionConfirmed: String = "EndEvent_SubscriptionConfirmed"
        val EndEvent_RegistrationCompleted: String = "EndEvent_RegistrationCompleted"
        val EndEvent_RegistrationNotPossible: String = "EndEvent_RegistrationNotPossible"
        val Activity_AbortRegistration: String = "Activity_AbortRegistration"
        val Activity_SendWelcomeMail: String = "Activity_SendWelcomeMail"
        val Activity_SendConfirmationMail: String = "Activity_SendConfirmationMail"
        val StartEvent_SubmitRegistrationForm: String = "StartEvent_SubmitRegistrationForm"
        val StartEvent_RequestReceived: String = "StartEvent_RequestReceived"
    }

    object Messages {
        val Message_FormSubmitted: String = "Message_FormSubmitted"
        val Message_SubscriptionConfirmed: String = "Message_SubscriptionConfirmed"
    }

    object TaskTypes {
        val EndEvent_RegistrationCompleted: String = "newsletter.registrationCompleted"
        val Activity_AbortRegistration: String = "newsletter.abortRegistration"
        val Activity_SendWelcomeMail: String = "newsletter.sendWelcomeMail"
        val Activity_SendConfirmationMail: String = "newsletter.sendConfirmationMail"
    }

    object Timers {
        val Timer_EveryDay: BpmnTimer = BpmnTimer("Duration", "PT1M")
        val Timer_After3Days: BpmnTimer = BpmnTimer("Duration", "PT2M30S")

        data class BpmnTimer(
            val type: String,
            val timerValue: String,
        )
    }

    object Errors {
        val Error_InvalidMail: BpmnError = BpmnError("Error_InvalidMail", "500")

        data class BpmnError(
            val name: String,
            val code: String,
        )
    }

    object Signals {
        val Signal_RegistrationNotPossible: String = "Signal_RegistrationNotPossible"
    }
}
```

Thanks to this generated Process API, there is no longer a need to rely on hardcoded strings. Instead, you can reference the variables provided in the API, effectively binding your code to the model. The technical configuration of the model becomes part of the codebase — in a type-safe, readable, and maintainable way. Let’s see how this changes our earlier examples.

Our worker now uses the Process API to reference the task type:

```kotlin
@Component
class AbortRegistrationWorker(
    private val useCase: AbortSubscriptionUseCase
) : DefaultJobWorker(
    type = NewsletterSubscriptionProcessApi.TaskTypes.Activity_AbortRegistration,
) {

    private val log = KotlinLogging.logger {}

    override fun executeTask(client: JobClient, job: ActivatedJob): Map<String, Any?> {
        val input = job.getVariablesAsType(Input::class.java)
        log.debug { "Received job to abort registration: $input" }
        useCase.abort(SubscriptionId(input.subscriptionId))
        return emptyMap()
    }

    data class Input(val subscriptionId: UUID)
}
```

Similarly, your tests can replace all string-based references to process elements with constants from the Process API.

```kotlin
@Test
fun `happy path - user subscribes to newsletter`() {

    val subscriptionId = UUID.fromString("4a607799-804b-43d1-8aa2-bdcc4dfd9b86")
    processPort.submitForm(SubscriptionId(subscriptionId))

    waitForProcessInstanceHasReachedElement(Activity_ConfirmRegistration)
    processPort.confirmSubscription(SubscriptionId(subscriptionId))

    waitForProcessInstanceHasPassedElement(EndEvent_RegistrationCompleted)
    assertThatProcess().isCompleted()
    assertThatProcess().hasPassedElementsInOrder(
        StartEvent_RequestReceived,
        Activity_SendConfirmationMail,
        Activity_ConfirmRegistration,
        Activity_SendWelcomeMail,
        EndEvent_RegistrationCompleted
    )

    verify { sendConfirmationMailUseCase.sendConfirmationMail(SubscriptionId(subscriptionId)) }
    verify { sendWelcomeMailUseCase.sendWelcomeMail(SubscriptionId(subscriptionId)) }
    verify { abortSubscriptionUseCase wasNot Called }
    confirmVerified(sendConfirmationMailUseCase, sendWelcomeMailUseCase, abortSubscriptionUseCase)
}
```

Be aware: All of this still won’t ensure that your code stays in sync when your model changes . However, it makes maintaining your solution much easier, because you only need to create a new version of the API based on the updated model and then adapt the affected areas.

### 🚀 Advantages

Now that we’ve explored what **bpmn-to-code** does and how it can be used in a real-world example, let’s step back and examine the key advantages this approach offers. These benefits highlight how the plugin can simplify your workflow and improve the quality of your automation projects:

- **Less manual work:** The plugin extracts all necessary API elements directly from the BPMN model, eliminating the tedious task of copy-pasting.
- **Faster time-to-market:** By automating the generation of code artifacts, you optimize development cycles & allow your team to focus on delivering robust solutions more quickly.
- **Fewer potential errors:** Automatically generating code artifacts significantly reduces the likelihood of typos or missed updates when models change.
- **Model-code consistency with versioning:** Since Process APIs can be regenerated anytime, your code stays effortlessly in sync with the model . Each API version receives a consecutive version number, making process changes traceable and clear.
- **Single source of truth:** Instead of having scattered strings throughout your codebase, you have one central Process API file per process. This file acts as the authoritative representation of your BPMN model, serving as a single reference point for your application.
- **Transparent processes encourage cleaner models:** With all process details available in code, readability and accessibility improve. This transparency helps spot inconsistencies and promotes better modeling practices, ultimately resulting in cleaner solutions.
- **Familiar approach:** The plugin uses a development approach similar to tools like Swagger, making it easier for newcomers to adopt process automation

### 🏁 Wrapping Up

Manually copying technical configurations from BPMN models into your code is not only annoying but also error-prone. It creates friction, especially as models evolve and diverge from the code. That’s the core problem I set out to solve. **bpmn-to-code** addresses these issues by generating **Process APIs** — code artifacts that represent your BPMN models directly in Kotlin or Java. With this approach, you can keep your code and models in sync, reduce manual effort, and create clean, robust, and developer-friendly automation solutions.

Thanks for reading! I’d love to hear your feedback and thoughts on how this could help in your projects.

*You can find the complete code & documentation to the plugin on [GitHub](https://github.com/Miragon/bpmn-to-code) if you’d like to explore it further.*