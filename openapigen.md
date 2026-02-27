# v01 

```
plugins {
    id 'org.springframework.boot' version '3.5.11'
    id 'io.spring.dependency-management' version '1.1.7'
    id 'de.undercouch.download' version '5.6.0'
    id 'org.openapi.generator' version '7.20.0'
    id 'com.diffplug.spotless' version '8.2.1'
	id 'idea'
}

tasks.named("openApiGenerate").configure { enabled = false }

group = 'com.example'
version = '0.0.1-SNAPSHOT'

repositories {
    mavenCentral()
}

ext {
    lombokVersion = "1.18.32"
    mapstructVersion = "1.6.3"
}

// -----------------------------
// Dependencies
// -----------------------------
dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter-web'
	
	// Dependencies commonly needed by generated models (e.g., for annotations)
    implementation 'io.swagger.api.v3:swagger-annotations:2.2.21' 
    implementation 'jakarta.validation:jakarta.validation-api:3.0.2'
    implementation 'com.fasterxml.jackson.api:jackson-annotations:2.15.3'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    // Lombok
    compileOnly "org.projectlombok:lombok:${lombokVersion}"
    annotationProcessor "org.projectlombok:lombok:${lombokVersion}"
    testCompileOnly "org.projectlombok:lombok:${lombokVersion}"
    testAnnotationProcessor "org.projectlombok:lombok:${lombokVersion}"

    // MapStruct + Lombok binding
    implementation "org.mapstruct:mapstruct:${mapstructVersion}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstructVersion}"
    annotationProcessor "org.projectlombok:lombok-mapstruct-binding:0.2.0"

    // Tests
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    componentTestImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// -----------------------------
// Java toolchain
// -----------------------------
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

// Ensure annotation processors are picked up
tasks.withType(JavaCompile).configureEach {
    options.annotationProcessorPath = configurations.annotationProcessor
    options.compilerArgs += ["-parameters"]
}

// -----------------------------
// Spotless
// -----------------------------
spotless {
    java {
        target(
            fileTree('src') { include '**/*.java' },
            fileTree(layout.buildDirectory.dir('generated/src/main/java')) { include '**/*.java' }
        )
        palantirJavaFormat()
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
        importOrder()
        formatAnnotations()
    }
}

// -----------------------------
// OpenAPI Model Generation
// -----------------------------
def specs = [
    [name: "a-api", specification: "a-integration.json",
     modelpackage: "com.example.api.a.integration.generated.model",
     url: "https://api-a-integration.apps.example.io/v3/api-docs"],

    [name: "b-api", specification: "b-integration.json",
     modelpackage: "com.example.api.b.integration.generated.model",
     url: "https://api-b-integration.apps.example.io/v3/api-docs"],

    [name: "c-api", specification: "c-integration.json",
     modelpackage: "com.example.api.c.integration.generated.model",
     url: "https://api-c-integration.apps.example.io/v3/api-docs"]
]

def openApiSpecDir = layout.buildDirectory.dir("openapi/specs")
def openApiOutputDir = layout.buildDirectory.dir("generated")

specs.each { spec ->
    def capitalizedName = spec.name.split('-').collect { it.capitalize() }.join('')
    def generateTaskName = "openApiGenerate${capitalizedName}"

    // OpenAPI generation task
    tasks.register(generateTaskName, org.openapitools.generator.gradle.plugin.tasks.GenerateTask) {
        generatorName = "spring"
        inputSpec = openApiSpecDir.map { it.file(spec.specification).asFile.absolutePath }.get()
        outputDir = openApiOutputDir.get().asFile.absolutePath

        modelPackage = spec.modelpackage
        validateSpec = false

        globalProperties = [
            models          : "",
            apis            : "false",
            supportingFiles : "false"
        ]

        configOptions = [
            useSpringBoot3          : "true",
            dateLibrary             : "java8",
            hideGenerationTimestamp : "true",
            useJakartaEe            : "true",
            generatedAnnotation     : "false",
			serializationLibrary	: "jackson",
            openApiNullable         : "false",
        ]
    }
}

// Aggregate OpenAPI generation
tasks.register("generateOpenApiModels") {
    dependsOn tasks.matching { it.name.startsWith("openApiGenerate") }
}

// -----------------------------
// Task dependencies
// -----------------------------
tasks.spotlessJava {
    dependsOn("generateOpenApiModels")
}

tasks.compileJava {
    dependsOn("spotlessApply")
}

// -----------------------------
// Source sets
// -----------------------------
sourceSets {
    main {
        java {
            srcDir("$buildDir/generated/src/main/java")
        }
    }
}

// -----------------------------
// Clean
// -----------------------------
tasks.clean {
    delete layout.buildDirectory.dir("generated")
}
```
-----

# v02
```
plugins {
    id 'org.springframework.boot' version '3.5.11'
    id 'io.spring.dependency-management' version '1.1.7'
    id 'de.undercouch.download' version '5.6.0'
    id 'org.openapi.generator' version '7.20.0'
    id 'com.diffplug.spotless' version '8.2.1'
    id 'governance-as-code' version '0.8.11'
    id 'idea'
}

tasks.named("openApiGenerate").configure { enabled = false }

group = 'com.example'
version = '0.0.1-SNAPSHOT'

repositories {
    mavenCentral()
}

ext {
    lombokVersion = "1.18.32"
    mapstructVersion = "1.6.3"
}

// -----------------------------
// Dependencies
// -----------------------------
dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // Dependencies commonly needed by generated models (e.g., for annotations)
    implementation 'io.swagger.core.v3:swagger-annotations:2.2.21' 
    implementation 'jakarta.validation:jakarta.validation-api:3.0.2'
    implementation 'com.fasterxml.jackson.core:jackson-annotations:2.15.3'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    // Lombok
    compileOnly "org.projectlombok:lombok:${lombokVersion}"
    annotationProcessor "org.projectlombok:lombok:${lombokVersion}"
    testCompileOnly "org.projectlombok:lombok:${lombokVersion}"
    testAnnotationProcessor "org.projectlombok:lombok:${lombokVersion}"

    // MapStruct + Lombok binding
    implementation "org.mapstruct:mapstruct:${mapstructVersion}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstructVersion}"
    annotationProcessor "org.projectlombok:lombok-mapstruct-binding:0.2.0"

    // Tests
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    componentTestImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// -----------------------------
// Java toolchain
// -----------------------------
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

// Ensure annotation processors are picked up
tasks.withType(JavaCompile).configureEach {
    options.annotationProcessorPath = configurations.annotationProcessor
    options.compilerArgs += ["-parameters"]
}

// -----------------------------
// Spotless
// -----------------------------
spotless {
    java {
        target(
            fileTree('src') { include '**/*.java' },
            fileTree(layout.buildDirectory.dir('generated/src/main/java')) { include '**/*.java' }
        )
        palantirJavaFormat()
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
        importOrder()
        formatAnnotations()
    }
}

// -----------------------------
// OpenAPI Model Generation
// -----------------------------
def specs = [
    [name: "a-api", specification: "a-integration.json",
     modelpackage: "com.example.api.a.integration.generated.model",
     url: "https://api-a-integration.apps.example.io/v3/api-docs"],

    [name: "b-api", specification: "b-integration.json",
     modelpackage: "com.example.api.b.integration.generated.model",
     url: "https://api-b-integration.apps.example.io/v3/api-docs"],

    [name: "c-api", specification: "c-integration.json",
     modelpackage: "com.example.api.c.integration.generated.model",
     url: "https://api-c-integration.apps.example.io/v3/api-docs"]
]

def openApiSpecDir = layout.buildDirectory.dir("openapi/specs")
def openApiOutputDir = layout.buildDirectory.dir("generated")

// Create the specs directory if it doesn't exist
tasks.register('createSpecsDir') {
    doLast {
        openApiSpecDir.get().asFile.mkdirs()
    }
}

specs.each { spec ->
    def capitalizedName = spec.name.split('-').collect { it.capitalize() }.join('')
    def downloadTaskName = "download${capitalizedName}Spec"
    def generateTaskName = "openApiGenerate${capitalizedName}"

    // Download task - THIS WAS MISSING
    tasks.register(downloadTaskName, de.undercouch.gradle.tasks.download.Download) {
        dependsOn 'createSpecsDir'
        src spec.url
        dest openApiSpecDir.map { it.file(spec.specification).asFile }.get()
        overwrite true
        // Optional: Add authentication if needed
        // username 'your-username'
        // password 'your-password'
    }

    // OpenAPI generation task
    tasks.register(generateTaskName, org.openapitools.generator.gradle.plugin.tasks.GenerateTask) {
        dependsOn downloadTaskName  // Now this dependency works
        generatorName = "spring"
        inputSpec = openApiSpecDir.map { it.file(spec.specification).asFile.absolutePath }.get()
        outputDir = openApiOutputDir.get().asFile.absolutePath

        modelPackage = spec.modelpackage
        validateSpec = false

        globalProperties = [
            models          : "",
            apis            : "false",
            supportingFiles : "false"
        ]

        configOptions = [
            useSpringBoot3          : "true",
            dateLibrary             : "java8",
            hideGenerationTimestamp : "true",
            useJakartaEe            : "true",
            generatedAnnotation     : "false",
            serializationLibrary    : "jackson",
            openApiNullable         : "false",
        ]
    }
}

// Aggregate OpenAPI generation
tasks.register("generateOpenApiModels") {
    dependsOn tasks.matching { it.name.startsWith("openApiGenerate") }
}

// -----------------------------
// IDE Configuration
// -----------------------------
idea {
    module {
        // Mark generated sources as such for IntelliJ
        generatedSourceDirs.add(file("$buildDir/generated/src/main/java"))
    }
}

// -----------------------------
// Task dependencies
// -----------------------------
tasks.spotlessJava {
    dependsOn("generateOpenApiModels")
}

tasks.compileJava {
    dependsOn("spotlessApply")
}

// Ensure generateOpenApiModels runs before spotlessJava
tasks.named('spotlessJava') {
    mustRunAfter('generateOpenApiModels')
}

// -----------------------------
// Source sets
// -----------------------------
sourceSets {
    main {
        java {
            srcDir("$buildDir/generated/src/main/java")
        }
    }
}

// -----------------------------
// Clean
// -----------------------------
tasks.clean {
    delete layout.buildDirectory.dir("generated")
    delete layout.buildDirectory.dir("openapi") // Also clean downloaded specs
}

// -----------------------------
// Optional: Add a task to download all specs without generating
// -----------------------------
tasks.register('downloadAllSpecs') {
    dependsOn tasks.matching { it.name.endsWith('Spec') && it.name.startsWith('download') }
}
```
#v03

```
plugins {
    id 'org.springframework.boot' version '3.5.11'
    id 'io.spring.dependency-management' version '1.1.7'
    id 'de.undercouch.download' version '5.6.0'
    id 'org.openapi.generator' version '7.20.0'
    id 'com.diffplug.spotless' version '8.2.1'
    id 'governance-as-code' version '0.8.11'
    id 'idea'
}

tasks.named("openApiGenerate").configure { enabled = false }

group = 'com.example'
version = '0.0.1-SNAPSHOT'

repositories {
    mavenCentral()
}

ext {
    lombokVersion = "1.18.32"
    mapstructVersion = "1.6.3"
}

// -----------------------------
// Dependencies
// -----------------------------
dependencies {
    // Spring Boot
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // Dependencies commonly needed by generated models (e.g., for annotations)
    implementation 'io.swagger.core.v3:swagger-annotations:2.2.21' 
    implementation 'jakarta.validation:jakarta.validation-api:3.0.2'
    implementation 'com.fasterxml.jackson.core:jackson-annotations:2.15.3'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    // Lombok
    compileOnly "org.projectlombok:lombok:${lombokVersion}"
    annotationProcessor "org.projectlombok:lombok:${lombokVersion}"
    testCompileOnly "org.projectlombok:lombok:${lombokVersion}"
    testAnnotationProcessor "org.projectlombok:lombok:${lombokVersion}"

    // MapStruct + Lombok binding
    implementation "org.mapstruct:mapstruct:${mapstructVersion}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstructVersion}"
    annotationProcessor "org.projectlombok:lombok-mapstruct-binding:0.2.0"

    // Tests
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    componentTestImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// -----------------------------
// Java toolchain
// -----------------------------
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

// Ensure annotation processors are picked up
tasks.withType(JavaCompile).configureEach {
    options.annotationProcessorPath = configurations.annotationProcessor
    options.compilerArgs += ["-parameters"]
}

// -----------------------------
// Spotless
// -----------------------------
spotless {
    java {
        target(
            fileTree('src') { include '**/*.java' },
            fileTree(layout.buildDirectory.dir('generated/src/main/java')) { include '**/*.java' }
        )
        palantirJavaFormat()
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
        importOrder()
        formatAnnotations()
    }
}

// -----------------------------
// OpenAPI Model Generation
// -----------------------------
def specs = [
    [name: "a-api", 
     specification: "openapi/specs/a-integration.json",  // Path relative to resources
     modelpackage: "com.example.core.a.integration.generated.model"],

    [name: "b-api", 
     specification: "openapi/specs/b-integration.json",
     modelpackage: "com.example.core.b.integration.generated.model"],

    [name: "c-api", 
     specification: "openapi/specs/c-integration.json",
     modelpackage: "com.example.core.c.integration.generated.model"]
]

def openApiOutputDir = layout.buildDirectory.dir("generated")

// Optional: Add validation task to ensure spec files exist
tasks.register('validateOpenApiSpecs') {
    doLast {
        def missingSpecs = []
        specs.each { spec ->
            def specFile = file("src/main/resources/${spec.specification}")
            if (!specFile.exists()) {
                missingSpecs.add("${spec.specification} for ${spec.name}")
            } else {
                println "Found spec: ${specFile}"
            }
        }
        
        if (!missingSpecs.isEmpty()) {
            throw new GradleException("Missing OpenAPI specifications:\n" + 
                missingSpecs.collect { "  - $it" }.join("\n") + 
                "\n\nPlease place your OpenAPI specs in src/main/resources/openapi/specs/")
        }
    }
}

specs.each { spec ->
    def capitalizedName = spec.name.split('-').collect { it.capitalize() }.join('')
    def generateTaskName = "openApiGenerate${capitalizedName}"

    // OpenAPI generation task
    tasks.register(generateTaskName, org.openapitools.generator.gradle.plugin.tasks.GenerateTask) {
        dependsOn 'validateOpenApiSpecs'  // Ensure specs exist before generating
        generatorName = "spring"
        inputSpec = file("src/main/resources/${spec.specification}").absolutePath
        outputDir = openApiOutputDir.get().asFile.absolutePath

        modelPackage = spec.modelpackage
        validateSpec = true  // Enable validation since we control the specs

        globalProperties = [
            models          : "",
            apis            : "false",
            supportingFiles : "false"
        ]

        configOptions = [
            useSpringBoot3          : "true",
            dateLibrary             : "java8",
            hideGenerationTimestamp : "true",
            useJakartaEe            : "true",
            generatedAnnotation     : "false",
            serializationLibrary    : "jackson",
            openApiNullable         : "false",
            // Optional: Add Lombok support
            additionalModelTypeAnnotations: "@lombok.Data @lombok.Builder @lombok.NoArgsConstructor @lombok.AllArgsConstructor"
        ]
    }
}

// Aggregate OpenAPI generation
tasks.register("generateOpenApiModels") {
    dependsOn tasks.matching { it.name.startsWith("openApiGenerate") }
}

// -----------------------------
// IDE Configuration
// -----------------------------
idea {
    module {
        // Mark generated sources as such for IntelliJ
        generatedSourceDirs.add(file("$buildDir/generated/src/main/java"))
        // Exclude the openapi specs directory from IntelliJ indexing if desired
        excludeDirs.add(file("src/main/resources/openapi"))
    }
}

// -----------------------------
// Task dependencies
// -----------------------------
tasks.spotlessJava {
    dependsOn("generateOpenApiModels")
}

tasks.compileJava {
    dependsOn("spotlessApply")
}

// Ensure generateOpenApiModels runs before spotlessJava
tasks.named('spotlessJava') {
    mustRunAfter('generateOpenApiModels')
}

// -----------------------------
// Source sets
// -----------------------------
sourceSets {
    main {
        java {
            srcDir("$buildDir/generated/src/main/java")
        }
        resources {
            // Ensure resources are available at runtime if needed
            srcDirs += ['src/main/resources']
        }
    }
}

// -----------------------------
// Copy specs to build directory (optional - for packaged applications)
// -----------------------------
tasks.named('processResources') {
    from('src/main/resources/openapi') {
        into('openapi')
    }
}

// -----------------------------
// Clean
// -----------------------------
tasks.clean {
    delete layout.buildDirectory.dir("generated")
}

// -----------------------------
// Optional: Task to list available specs
// -----------------------------
tasks.register('listOpenApiSpecs') {
    dependsOn 'validateOpenApiSpecs'
    doLast {
        println "\nAvailable OpenAPI Specifications:"
        println "=================================="
        specs.each { spec ->
            def specFile = file("src/main/resources/${spec.specification}")
            println "${spec.name}:"
            println "  File: ${spec.specification}"
            println "  Package: ${spec.modelpackage}"
            println "  Size: ${specFile.length()} bytes"
            println "  Last Modified: ${new Date(specFile.lastModified())}"
            println()
        }
    }
}
```

#
