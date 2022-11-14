---
title: "Second Blog"
date: 2022-11-14T12:20:00+05:30
---

Gradle is one of the important files that are present in every Android application project. We use build.gradle file to manage all the dependencies that are required in our application. But all the code inside the Gradle file is written in Groovy programming language and nowadays, we write the code of our application in Kotlin because of its concise, null-safe, and interoperable nature. So, how about using the same Kotlin language for dependency management in our Android application project? Things will be much easier, right?

So, in this blog, we will learn how to migrate our Android application project from a Groovy-based Gradle system to a Kotlin-based Gradle system with the help of Kotlin DSL. Here is what we are going to learn in this blog:

How to migrate to Kotlin based Gradle?
How to have a central dependency management system?
What are the advantages and disadvantages of migrating to Kotlin based Gradle?
So, let's get migrated :)

How to migrate to Kotlin based Gradle?
From the official documentation:

Gradle’s Kotlin DSL provides an alternative syntax to the traditional Groovy DSL with an enhanced editing experience in supported IDEs, with superior content assist, refactoring, documentation, and more.
So, in short, we can leverage the functionality of Kotlin and the code suggestion feature along with other features of the IDEs in our Gradle file also just like any other normal Kotlin file. So, let's perform our first step of migration.

Replacing a single quote with double-quote
So, the very first thing that you need to do is changing all the single quotes( ' ' ) to double quotes( " " ) in your existing build.gradle file. This is because a Groovy string can be inside a single quote as well as inside a double quote but in Kotlin, only double quotes are allowed for a string. For example:

// before
implementation 'com.mindorks.android:prdownloader:$prdownloader_version'
// after
implementation "com.mindorks.android:prdownloader:$prdownloader_version"
Using function calls and = assignment operator
In Groovy, while calling a function, there is no need for parentheses but in Kotlin, it is required to call the function by using the parentheses.

Also, in Groovy, while assigning some value to a variable you can skip the use of = assignment operator but in Kotlin, = assignment operator is a must.

So, our next step is to replace all the function calls by using the parentheses and all the direct assignments by using the = operator. But in a Gradle file, it is difficult to identify between function call and assignment. So, if you are not sure about these two, then what you can do here is use the function call in all places and if an error comes then simply replace it with the = assignment operator. So, here are a few examples for the use of function call and the = assignment operator.

// example of function call
implementation("com.mindorks.android:prdownloader:$prdownloader_version")
// example of = assignment
defaultConfig {
    versionName = 3
}
Change filename
Now, we are done with our basic setup. It's time to change the file name because we are going to use Kotlin DSL. So, name all you .gradle file to .gradle.kts file. For example, build.gradle will become build.gradle.kts.

Using Plugin
To use plugins in your Gradle file, use the following:

plugins {
    id("com.google.gms.google-services")
}
Or if you are using the legacy apply( ) function, then you can follow the below approach:

apply(plugin = "com.google.gms.google-services")
Using URLs
In the Gradle file, you can use the URL like this:

// before
maven { url 'https://jitpack.io' }
// after
maven("https://jitpack.io")
Using tasks
Now, to register your task, you need to do this:

// before
task clean(type: Delete)  {
    delete rootProject.buildDir
}
// after
tasks.register("clean",  Delete::class)  {
    delete(rootProject.buildDir)
}
Using Boolean variables
To use a boolean variable, you need to use "is" as a prefix to the variable. But it is always a good practice to use "is" as a prefix for boolean variables in all programming languages.

// before
debuggable false
// after
isDebuggable = false
Using getByName for buildTypes
To use various build types, you need to use the getByName function:

// before
buildTypes {
    debug  {
        // ...
    }
    release {  
        // ...
    }
}
// after
buildTypes {
    getByName("debug") {
        // ...
    }
    getByName("release") {
        // ...
    }
}
Mapping using mapOf
To map variables, you need to use the mapOf() function. For example, to add local .jar file dependency to your build.gradle.kts file, you need to use:

// before
implementation fileTree(dir: 'libs', include: ['*.jar'])
// after
implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
Using variables from project-level Gradle file(root project)
For better dependency management, many developers tend to put the versions of all the dependencies in the root-level build.gradle file and use it in the app-level build.gradle file. So, we can use this like below:

// in root-level build.gradle file
val prdownloaderVersion by rootProject.extra("0.6.0")
// in app-level build.gradle file
implementation("com.mindorks.android:prdownloader:${rootProject.extra.get("prdownloaderVersion")}")
Excluding libraries from all dependencies
To exclude some libraries from all dependencies in Kotlin DSL build.gradle file, use the following:

configurations {
    all {
        exclude(group = "com.google.guava", module = "listenablefuture")
    }
}
These are some steps that can be used to migrate from a Groovy-based Gradle file to a Kotlin-based Gradle file.

Thumbnail Image
30% DISCOUNT NOW
Android Online Course for Professionals by MindOrks
Join and learn Dagger, Kotlin, RxJava, MVVM, Architecture Components, Coroutines, Unit Testing and much more.

Now, it will be quite easier for you to manage the code because now you must be getting auto suggestions, code navigation, and obviously all the awesome features of the Kotlin language.

But things will be much easier if you have a central dependency management system for your whole app. So, let's learn how to make one.

How to have a central dependency management system?
Whenever we run any Gradle file, then it checks for buildSrc and based on that it compiles the code. So, we can take the advantage of this feature/process and can make our dependency files written in Kotlin language.

So, in the project view, create a new directory named buildSrc in your root directory, and inside the buildSrc directory, make a file name build.gradle.kts and add the below code:

repositories {
    jcenter()
}

plugins {
    `kotlin-dsl`
}
You can remove the repositories { } section based on the requirement of the project. Now, sync the project.

Now, in the buildSrc directory, create nested directories by entering the name as src/main/java. You can create these directories one by one also. Inside buildSrc/src/main/java, create a new file named Dependencies.kt.

Inside this Dependencies.kt file, we are going to make objects for different purposes like we can have one object for all the versions, also, we can have one object for all Dependencies, and so on.

For example:

object Versions {
    const val recyclerViewVersion = "1.1.0"
    const val materialLibraryVersion = "1.2.1"
}
object Dependencies {
    const val recyclerView = "androidx.recyclerview:recyclerview:${Versions.recyclerViewVersion}"
    const val materialDesign =   "com.google.android.material:material:${Versions.materialLibraryVersion}"
}
Similarly, you can have other objects also like one object for your default configuration and so on.

Now, to use these dependencies in your app-level build.gradle file, all you need to do is call it by its name. For example, here is the code of the app-level build.gradle file:

dependencies {
    implementation(Dependencies.materialDesign)

    implementation(Dependencies.recyclerView)
}
In this way, you can keep all the dependencies and its version in one place and this code is quite easy to read and maintain.

We have seen how to migrate from Groovy Gradle to Kotlin Gradle, but there are some disadvantages also of this process. Let's find out.

What are the advantages and disadvantages of migrating to Kotlin based Gradle?
Till now, we have seen what are the advantages of using Kotlin DSL in our Gradle file. So, let's summarize it:

Since you are using Kotlin, then the IDE will provide the auto-suggestion feature for your Gradle file also which is missing till now.
Code navigation between files becomes easy in Kotlin DSL.
You can leverage all the functionalities of Kotlin in your Gradle file.
But, there are a few disadvantages also. Here are some:

In some cases, Kotlin DSL might be slower than the Groovy one. For example, it can be slower in case of first use, on clean checkouts, or on ephemeral CI agents.
If you are migrating from Groovy to DSL, then things might look difficult and lengthy for you, but it is worth doing.
Another disadvantage or you can say prerequisite for using Kotlin DSL in Gradle is that you must run your Gradle with Java 8 or higher versions.
So, these are some of the advantages and disadvantages of using Kotlin DSL for your Gradle files and it is clear that the advantages are quite more than that of disadvantages.

We are quite sure that you are going to use and love Kotlin DSL for your Gradle files.

That's it for this blog. Hope you enjoyed the migration process.

Show your love by sharing this blog with your fellow developers.

MindOrks!

Happy Learning :)