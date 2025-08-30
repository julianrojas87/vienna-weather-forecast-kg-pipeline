# vienna-weather-forecast-kg-pipeline

RDF-Connect pipeline to produce a knowledge graph from Vienna’s weather forecast data. 
This repository provides **incremental solutions** for the hands-on tutorial at 
[SEMANTiCS 2025](https://2025-eu.semantics.cc/page/cfp_ws):  
👉 [Tutorial Website](https://rdf-connect.github.io/Tutorial-SEMANTiCS2025/) & [Tutorial Slides](https://rdf-connect.github.io/Tutorial-SEMANTiCS2025/slides)

🌐 [RDF-Connect Homepage](https://rdf-connect.github.io) | [RDF-Connect GitHub](https://github.com/rdf-connect)

---

## RDF-Connect Tutorial

This tutorial walks you step by step through building a **provenance-aware, streaming RDF pipeline** using the language-agnostic framework **RDF-Connect**.  

The use case: Producing and publishing a queryable **knowledge graph** from Vienna's weather forecast data extracted from the **GeoSphere Austria API**.

You will:

- Set up an RDF-Connect environment  
- Configure pipeline components  
- Implement processors in multiple programming languages  
- Run the pipeline end-to-end

By the end, you will have:

- A working RDF-Connect pipeline for real-world data  
- A clear understanding of how to integrate heterogeneous processors across execution environments 
- Practical experience with implementing RDF-Connect processors

The tutorial is designed for all experience levels, and you can follow along at your own pace.  
Each **task** builds on the previous one, and each solution is available in a dedicated **branch** of this repository (`task-1`, `task-2`, ...).  
You can use these branches to verify your work, catch up if stuck, or compare with the reference solution.

---

## Getting Started

The recommended starting point is to **fork and clone this repository**, then switch to the `main` branch.

### Prerequisites

Make sure the following are installed:

- **Node.js ≥16**  
- **Java ≥17**  
  - Gradle ≥8.5 (you can also manually download the JARs and put them in `pipeline/build/plugins/`)
- **Python ≥3.8** (we recommend 3.13 for Part 2)  
  - Hatch (for managing Python environments and dependencies)
  - uv (for managing Python packages)

**If you do not want to install these tools locally**, we have provided a **Dockerfile** that sets up an environment with all software installed.
You can build and run it with the following commands:

```bash
# Build the Docker image
cd pipeline/resources
docker build -t devbox .

cd pipeline/  # or `cd ..` to go back to the pipeline directory

# Run a container with an interactive shell
docker run -it --rm -v $(pwd):/workspace devbox
# You can now run commands like `npm install` or `npx rdfc pipeline.ttl` inside the container

# Or directly run the commands
docker run --rm -v $(pwd):/workspace devbox npm install
docker run --rm -v $(pwd):/workspace devbox npx rdfc pipeline.ttl
```

The pipeline will store data in a **Virtuoso triple store**.  
We recommend running Virtuoso via **Docker + Docker Compose**, so install both if you plan to follow that setup.
You can also use your own Virtuoso instance if you prefer.

---

## Tasks

### Part 1: Assembling the Pipeline

#### Task 0: Set up the project structure for the pipeline

In this step, you’ll prepare the project with an empty pipeline config.  
You may start from our provided project structure (recommended) or consult the [example pipelines](https://github.com/rdf-connect/example-pipelines/tree/main/java/hello-world) repository for inspiration.

**Steps:**

- [x] Create a `pipeline/` directory (all Part 1 work happens here).  
- [x] Inside `pipeline/`, create:
  - `pipeline.ttl` (pipeline configuration)  
  - `README.md` (documentation)  
  - `package.json` (via `npm init` or manually)  
  - `.gitignore` (exclude `node_modules/` etc.)  
- [x] Install the orchestrator:  
  ```bash
  npm install @rdfc/orchestrator-js
  ```
- [ ] Install the Javascript runner:
  ```bash
  npm install @rdfc/js-runner
  ```

  **Expected folder structure:**
  ```
  ├── pipeline/           # Part 1 work lives here
  │   ├── node_modules/   
  │   ├── .gitignore      
  │   ├── package.json    
  │   ├── pipeline.ttl    
  │   └── README.md       
  ├── processor/          # Custom processor (Part 2)
  └── README.md           # Tutorial instructions
  ```

- [ ] Initialize the RDF-Connect pipeline in `pipeline.ttl`:
  - Add RDF namespaces (e.g., `rdfc`, `owl`, `ex`)
  - Declare the pipeline with the following triple:
    ```turtle
    <> a rdfc:Pipeline.
    ```

  **Expected pipeline state:**
  ```turtle
  @prefix rdfc: <https://w3id.org/rdf-connect#>.
  @prefix owl: <http://www.w3.org/2002/07/owl#>.
  @prefix ex: <http://example.org/>.

  ### Import runners and processors

  ### Define the channels

  ### Define the pipeline
  <> a rdfc:Pipeline.

  ### Define the processors
  ```

✅ The solution for this task is in the **`main` branch**.


#### Task 1: Fetch weather data from the GeoSphere Austria API

Configure the pipeline to fetch weather data from GeoSphere Austria (station `11035`, near the SEMANTiCS venue) in JSON format:

**API endpoint**:  
- <https://dataset.api.hub.geosphere.at/v1/station/current/tawes-v1-10min?parameters=TL,RR&station_ids=11035>

**Processors to add:**

- `rdfc:HttpFetch` – HTTP processor implemented in TypeScript (implementation & documentation at [@rdfc/http-utils-processor-ts](https://github.com/rdf-connect/http-utils-processor-ts))
- `rdfc:LogProcessorJs` – Processor that logs to RDF-Connect logging system any input stream, implemented in TypeScript (implementation & documentation at [@rdfc/log-processor-ts](https://github.com/rdf-connect/log-processor-ts))

**Runners to add:**

- `rdfc:NodeRunner` – run JavaScript processors (implementation & documentation at [@rdfc/js-runner](https://github.com/rdf-connect/js-runner))

**Steps:**

- [ ] Add an `rdfc:HttpFetch` processor instance
  - Install the processor
    ```bash
    npm install @rdfc/http-utils-processor-ts
    ```
  - Import semantic definition via `owl:imports`  
    ```turtle
    ### Import runners and processors
    <> owl:imports <./node_modules/@rdfc/http-utils-processor-ts/processors.ttl>.
    ```  
  - Define a channel for the fetched JSON data
    ```turtle
    ### Define the channels
    <json> a rdfc:Reader, rdfc:Writer.
    ```
  - Configure it to fetch from the API endpoint  
    ```turtle
    ### Define the processors
    # Processor to fetch data from a JSON API
    <fetcher> a rdfc:HttpFetch;
        rdfc:url <https://dataset.api.hub.geosphere.at/v1/station/current/tawes-v1-10min?parameters=TL,RR&station_ids=11035>;
        rdfc:writer <json>.
    ```
- [ ] Add an `rdfc:NodeRunner` Node.js runner instance
  - Install the runner
    ```bash
    npm install @rdfc/js-runner
    ```
  - Import its semantic definition  
    ```turtle
    ### Import runners and processors
    <> owl:imports <./node_modules/@rdfc/js-runner/index.ttl>.
    ```
  - Define it as part of the pipeline and link the `rdfc:HttpFetch` processor instance to it using the `rdfc:consistsOf`,  `rdfc:instantiates` and `rdfc:processor` properties
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
     rdfc:consistsOf [
         rdfc:instantiates rdfc:NodeRunner;
         rdfc:processor <fetcher>;
     ].
    ```
- [ ] Add a `rdfc:LogProcessorJs` processor instance
  - Install the processor
    ```bash
    npm install @rdfc/log-processor-ts
    ```
  - Import its semantic definition
    ```turtle
    ### Import runners and processors
    <> owl:imports <./node_modules/@rdfc/log-processor-ts/processor.ttl> .
    ```
  - Create an instance and configure it with e.g., log level: `info`, label: `output` and link it to the output channel of `rdfc:HttpFetch`
    ```turtle
    ### Define the processors
    # Processor to log the output
    <logger> a rdfc:LogProcessorJs;
          rdfc:reader <json>;
          rdfc:level "info";
          rdfc:label "output".
    ``` 
  - Attach it to the `rdfc:NodeRunner`
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
       rdfc:consistsOf [
           rdfc:instantiates rdfc:NodeRunner;
           rdfc:processor <fetcher>, <logger>;
       ].
    ```
- [ ] Run the pipeline:  
  ```bash
  npx rdfc pipeline.ttl
  # or with debug logging:
  LOG_LEVEL=debug npx rdfc pipeline.ttl
  ```

✅ Complete solution available in **`task-1` branch**.


#### Task 2: Convert the weather data from JSON to RDF

You will now convert the JSON stream into RDF using **[RML](https://rml.io/)** with the help of the `rdfc:RmlMapper` processor.

To help you with this, we prepared an [RML mapping file](./pipeline/resources/mapping.rml.ttl) for you that you can use to convert the JSON data to RDF.

**Processors to add:**

- `rdfc:GlobRead` – read mapping file from disk, implemented in TypeScript (implementation & documentation at [@rdfc/file-utils-processors-ts](https://github.com/rdf-connect/file-utils-processors-ts))  
- `rdfc:RmlMapper` – convert heterogeneous data to RDF, implemented in Java (implementation & documentation at [rml-processor-jvm](https://github.com/rdf-connect/rml-processor-jvm)). Internally, it uses the [RMLMapper engine](https://github.com/RMLio/rmlmapper-java)

**Runners to add:**

- `rdfc:JvmRunner` – run Java processors (implementation & documentation at [rdf-connect/jvm-runner](https://github.com/rdf-connect/jvm-runner))  

**Steps:**

- [ ] Use `rdfc:GlobRead` to read the RML mapping file
  - Install this Node.js processor
    ```bash
    npm install @rdfc/file-utils-processors-ts
    ```
  - Import its semantic definition into the pipeline
    ```turtle
    ### Import runners and processors
    <> owl:imports <./node_modules/@rdfc/file-utils-processors-ts/processors.ttl>.
    ```
  - Define a channel for the RML mapping data
    ```turtle
    ### Define the channels
    <mappingData> a rdfc:Reader, rdfc:Writer.
    ```
  - Create an instance and configure it to read the mapping file from disk (e.g., `./resources/mapping.rml.ttl`)
    ```turtle
    ### Define the processors
    # Processor to read and stream out the RML mappings
    <mappingReader> a rdfc:GlobRead;
        rdfc:glob <./resources/mapping.rml.ttl>;
        rdfc:output <mappingData>;
        rdfc:closeOnEnd false.
    ```
  - Attach it to the existing `rdfc:NodeRunner`
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
       rdfc:consistsOf [
           rdfc:instantiates rdfc:NodeRunner;
           rdfc:processor <fetcher>, <logger>, <mappingReader>;
       ].
    ```
- [ ] Add a Java Virtual Machine (JVM) runner (`rdfc:JvmRunner`) that allow us to execute Java processors
  - Import its semantic definition which, in this case, is packed within the built JAR file of the runner
    ```turtle
    ### Import runners and processors
    <> owl:imports <https://javadoc.jitpack.io/com/github/rdf-connect/jvm-runner/runner/master-SNAPSHOT/runner-master-SNAPSHOT-index.jar>.
    ```
  - Link it to the pipeline
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
       rdfc:consistsOf [
           rdfc:instantiates rdfc:NodeRunner;
           rdfc:processor <fetcher>, <logger>, <mappingReader>;
       ], [
           rdfc:instantiaties rdfc:JvmRunner;
       ].
    ```
- [ ] Add an `rdfc:RmlMapper` processor instance
  - Install the Java processor using Gradle:
    - If you do not want to use Gradle, you can manually download the JAR files from [JitPack](https://javadoc.jitpack.io/com/github/rdf-connect/rml-processor-jvm/master-SNAPSHOT/rml-processor-jvm-master-SNAPSHOT-all.jar) and put them in `pipeline/build/plugins/`. Otherwise, you can use the provided Dockerfile to run Gradle in a container.
    - Create a `build.gradle` file inside the `./pipeline` folder with the following content
        ```gradle
        plugins {
            id 'java'
        }
    
        repositories {
            mavenCentral()
            maven { url = uri("https://jitpack.io") }  // if your processors are on GitHub
        }
        dependencies {
            implementation("com.github.rdf-connect:rml-processor-jvm:master-SNAPSHOT:all")
        }
    
        tasks.register('copyPlugins', Copy) {
            from configurations.runtimeClasspath
            into "$buildDir/plugins"
        }
    
        configurations.all {
            resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
        }
        ```
    - Build and pack the processor binary
      ```bash
      gradle copyPlugins
      ```
    - Import its semantic definition
      ```turtle
      ### Import runners and processors
      <> owl:imports <./build/plugins/rml-processor-jvm-master-SNAPSHOT-all.jar>.
      ```
    - Define an output channel for the resulting RDF data
      ```turtle
      ### Define the channels
      <rdf> a rdfc:Reader, rdfc:Writer.
      ```
    - Create an instance (`rdfc:RmlMapper`) and configure it to receive the RML mapping rules and JSON data stream
      ```turtle
      ### Define the processors
      # Processor to do the RML mapping
      <mapper> a rdfc:RmlMapper;
          rdfc:mappings <mappingData>;
          rdfc:source [
              rdfc:triggers true;
              rdfc:reader <json>;
              rdfc:mappingId ex:source1;
          ];
          rdfc:defaultTarget [
              rdfc:writer <rdf>;
              rdfc:format "turtle";
          ].
      ```
    - Link the processor to the corresponding runner using the `rdfc:processor` property
      ```turtle
      ### Define the pipeline
      <> a rdfc:Pipeline;
         rdfc:consistsOf [
             rdfc:instantiates rdfc:NodeRunner;
             rdfc:processor <fetcher>, <logger>, <mappingReader>;
         ], [
             rdfc:instantiaties rdfc:JvmRunner;
             rdfc:processor <mapper>;
         ].
      ```
- [ ] Redirect the logging processor to log the resulting **RDF output** instead of the initial raw JSON
  ```turtle
  ### Define the processors
  # Processor to log the output
  <logger> a rdfc:LogProcessorJs;
        rdfc:reader <rdf>;
        rdfc:level "info";
        rdfc:label "output".
  ```
- [ ] Run the pipeline:  
  ```bash
  npx rdfc pipeline.ttl
  # or with debug logging:
  LOG_LEVEL=debug npx rdfc pipeline.ttl
  ```

✅ Complete solution available in **`task-2` branch**.


#### Task 3: Validate the produced RDF with SHACL

Next, validate the RDF output against a provided SHACL shape.

To help you with this, we prepared a [SHACL shape file](./pipeline/resources/shacl-shape.ttl) that you can use to validate the RDF data.


**Processors to add:**

- `rdfc:Validate` – validate RDF data using a given SHACL shape, implemented in TypeScript (implementation & documentation at [@rdfc/shacl-processor-ts](https://github.com/rdf-connect/shacl-processor-ts)). Internally, this processor relies on [`shacl-engine`](https://github.com/rdf-ext/shacl-engine), a JavaScript SHACL engine implementation  
- Another instance of `rdfc:LogProcessorJs` – for logging SHACL validation reports  

**Steps:**

- [ ] Add an `rdfc:Validate` processor instance
  - Install the processor
    ```bash
    npm install @rdfc/shacl-processor-ts
    ```
  - Import its semantic definition into the pipeline
    ```turtle
    ### Import runners and processors
    <> owl:imports <./node_modules/@rdfc/shacl-processor-ts/processors.ttl>.
    ```
  - Define a channel for the SHACL validation reports and for the successfully validated RDF data
    ```turtle
    ### Define the channels
    <report> a rdfc:Reader, rdfc:Writer.
    <validated> a rdfc:Reader, rdfc:Writer.
    ```
  - Create an instance and configure it to use the provided SHACL shape file and to read the stream of produced RDF data
    ```turtle
    ### Define the processors
    # Processor to validate the output RDF with SHACL
    <validator> a rdfc:Validate;
        rdfc:shaclPath <./resources/shacl-shape.ttl>;
        rdfc:incoming <rdf>;
        rdfc:outgoing <validated>;
        rdfc:report <report>;
        rdfc:validationIsFatal false;
        rdfc:mime "text/turtle".
    ```
  - Link it to the corresponding runner: `rdfc:NodeRunner`
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
       rdfc:consistsOf [
           rdfc:instantiates rdfc:NodeRunner;
           rdfc:processor <fetcher>, <logger>, <mappingReader>, <validator>;
       ], [
           rdfc:instantiaties rdfc:JvmRunner;
           rdfc:processor <mapper>;
       ].
    ```
- [ ] Use a new instance of `rdfc:LogProcessorJs` to log validation reports at `warn` level
  - Define the new logger instance
    ```turtle
    ### Define the processors  
    # Processor to log the SHACL report
    <reporter> a rdfc:LogProcessorJs;
        rdfc:reader <report>;
        rdfc:level "warn";
        rdfc:label "report".
    ```
  - Link it to the corresponding runner: `rdfc:NodeRunner`
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
       rdfc:consistsOf [
           rdfc:instantiates rdfc:NodeRunner;
           rdfc:processor <fetcher>, <logger>, <mappingReader>, <validator>, <reporter>;
       ], [
           rdfc:instantiaties rdfc:JvmRunner;
           rdfc:processor <mapper>;
       ].
    ```
- [ ] Log only valid data through the first logger
  ```turtle
  ### Define the processors
  # Processor to log the output
  <logger> a rdfc:LogProcessorJs;
      rdfc:reader <validated>;  # update the channel it logs
      rdfc:level "info";
      rdfc:label "output".
  ``` 
- [ ] Run the pipeline with a successfully validated result. You shall see the produced RDF in the console, similarly to the outcome of `task-2`, given that the validation is successful.
  ```bash
  npx rdfc pipeline.ttl
  ```
- [ ] Run the pipeline with a failed validation
  - To see the validation process in action, let's alter the SHACL shape to require a property that won't be present in the data. We can add the following property shape
    ```turtle
    ex:ObservationCollectionShape a sh:NodeShape ;
        #...
        sh:property [
            sh:path sosa:fakeProperty ;
            sh:class sosa:Observation ;
            sh:minCount 1 ;
        ] .
    ```
  - Run the pipeline again to see the warning report
    ```bash
    npx rdfc pipeline.ttl
    ```

✅ Complete solution available in **`task-3` branch**.


#### Task 4: Ingest validated RDF weather data into Virtuoso triple store

Finally, ingest the validated data into a **Virtuoso triple store** (via Docker Compose, or your own instance).

To help you with this, we prepared a [Docker Compose file](./pipeline/resources/docker-compose.yml) that you can use to run a Virtuoso instance via Docker.
The instance provided in the Docker Compose file is configured to be accessible at `http://localhost:8890/sparql` with SPARQL update enabled.

**Processors to add:**

- `rdfc:Sdsify` – convert RDF to SDS records, implemented in TypeScript (implementation & documentation at [@rdfc/sds-processors-ts](https://github.com/rdf-connect/sds-processors-ts))  
- `rdfc:SPARQLIngest` – send SDS records to Virtuoso, implemented in TypeScript (implementation & documentation at [@rdfc/sparql-ingest-processor-ts](https://github.com/rdf-connect/sparql-ingest-processor-ts))  

**Steps:**

- [ ] Convert validated RDF to SDS records with `rdfc:Sdsify`
  - Install the processor
    ```bash
    npm install @rdfc/sds-processors-ts
    ```
  - Import its semantic definition into the pipeline
    ```turtle
    ### Import runners and processors
    <> owl:imports <./node_modules/@rdfc/sds-processors-ts/configs/sdsify.ttl>.
    ```
  - Define a channel for the SDS records
    ```turtle
    ### Define the channels
    <sdsified> a rdfc:Reader, rdfc:Writer.
    ```
  - Create an instance and configure it to read the validated RDF data and output SDS records, and set an SDS stream ID (e.g., `http://ex.org/ViennaWeather`)
    ```turtle
      ### Define the processors
      # Processor to describe every entity as part of a SDS stream for further processing
      <sdsifier> a rdfc:Sdsify;
          rdfc:input <validated>;
          rdfc:output <sdsified>;
          rdfc:stream <http://ex.org/ViennaWeather>.
    ```
  - Link it to the corresponding runner: `rdfc:NodeRunner`
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
       rdfc:consistsOf [
           rdfc:instantiates rdfc:NodeRunner;
           rdfc:processor <fetcher>, <logger>, <mappingReader>, <validator>, <reporter>, <sdsifier>;
       ], [
           rdfc:instantiaties rdfc:JvmRunner;
           rdfc:processor <mapper>;
       ].
    ```
- [ ] Add the `rdfc:SPARQLIngest` processor instance to ingest SDS records into the Virtuoso instance
  - Install the processor
    ```bash
    npm install @rdfc/sparql-ingest-processor-ts
    ```
  - Import its semantic definition into the pipeline
    ```turtle
    ### Import runners and processors
    <> owl:imports <./node_modules/@rdfc/sparql-ingest-processor-ts/processors.ttl>.
    ```
  - Define a channel for the SPARQL queries sent to Virtuoso
    ```turtle
    ### Define the channels
    <sparql> a rdfc:Reader, rdfc:Writer.
    ```
  - Create an instance and configure it to read the SDS records and send them to the Virtuoso SPARQL endpoint
    ```turtle
    ### Define the processors
    # Processor to ingest SDS records into a SPARQL endpoint
    <ingester> a rdfc:SPARQLIngest;
        rdfc:memberStream <sdsified>;
        rdfc:ingestConfig [
            rdfc:memberIsGraph false;
            rdfc:targetNamedGraph "http://ex.org/ViennaWeather";
            rdfc:graphStoreUrl "http://localhost:8890/sparql";
            rdfc:forVirtuoso true
        ];
        rdfc:sparqlWriter <sparql>.
    ```
  - Link it to the corresponding runner: `rdfc:NodeRunner`
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
       rdfc:consistsOf [
           rdfc:instantiates rdfc:NodeRunner;
           rdfc:processor <fetcher>, <logger>, <mappingReader>, <validator>, <reporter>, <sdsifier>, <ingester>;
       ], [
           rdfc:instantiaties rdfc:JvmRunner;
           rdfc:processor <mapper>;
       ].
    ```
- [ ] Change the input channel of the first `rdfc:LogProcessorJs` processor to the output channel of the `rdfc:SPARQLIngest` processor to log the SPARQL queries that are sent to the Virtuoso instance.
- [ ] Start the Virtuoso instance via Docker Compose (if you haven't already)
  ```bash
  cd resources
  docker-compose up -d
  ```
- [ ] Run the pipeline:  
  ```bash
  npx rdfc pipeline.ttl
  # or with debug logging:
  LOG_LEVEL=debug npx rdfc pipeline.ttl
  ```

✅ Complete solution available in **`task-4` branch**.  

🎉 You have now completed **Part 1**! Your pipeline fetches, converts, validates, and ingests Vienna’s weather forecast into Virtuoso. You can query the data using SPARQL.


---

### Part 2: Implementing a Custom Processor

The data includes **German literals** (`@de`). To make it more accessible, we will implement a **custom Python processor** that translates them into English (`@en`) using a lightweight local ML model from Hugging Face.

#### Task 5: Set up the processor project

As you might have noticed, we have worked in the `pipeline/` directory for the first part of the tutorial.
However, there is also a `processor/` directory in the root of the project.
This is where you will implement the custom Python processor in this part of the tutorial.

To kickstart the development of a new processor, the RDF-Connect ecosystem provides template repositories that you can use as a starting point, allowing you to directly dive into the actual processor logic without having to worry about the project setup and configuration.
We will use the [template-processor-py](https://github.com/rdf-connect/template-processor-py) repository as a starting point.

**Steps:**

- [ ] Either clone the template or use the preconfigured project in `processor/`  
- [ ] Install dependencies (see template `README.md`)  
- [ ] Rename the template processor (e.g., `TranslationProcessor`) in `processor.py`, `processor.ttl`, `pyproject.toml`, and `README.md`
  - See "Next Steps" in the `README.md` file of the template repository for guidance.
- [ ] Build and verify  

✅ Complete solution available in **`task-5` branch**


#### Task 6: Implement translation logic

We’ll translate German literals using the Hugging Face model [Helsinki-NLP/opus-mt-de-en](https://huggingface.co/Helsinki-NLP/opus-mt-de-en).

**Steps:**

- [ ] Install `transformers` and its dependencies and the `rdflib` library for RDF parsing:
  ```bash
  uv add transformers sacremoses sentencepiece torch rdflib
  ```  
- [ ] Load the model + tokenizer in `TranslationProcessor.init`  
- [ ] In `transform`, implement the logic to translate language-tagged literals:
  - parse triples with `rdflib`  
  - Identify literals with `@de`  
  - Translate to English  
  - Emit both original and translated triples  
- [ ] (Optional) Add unit tests  
- [ ] Build the project  

✅ Complete solution available in **`task-6` branch**


#### Task 7: Integrate the processor into the pipeline

Run your Python processor inside the pipeline with a Python runner for RDF-Connect.

**Runners to add:**
- `rdfc:PyRunner` — run Python processors (implementation & documentation at [rdf-connect/py-runner](https://github.com/rdf-connect/py-runner)).

**Steps:**

- [ ] Build the processor into a package
  ```bash
  hatch build
  ```
- [ ] Create a `pyproject.toml` file inside the `pipeline/` folder to configure the Python environment for the pipeline
  - Specify the Python version to use to one specific version (e.g., `==3.13.*`). You need this to have a deterministic path for the `owl:imports` statement
  - Configure `[tool.hatch.envs.default]` to use a virtual environment called `.venv`
  ```toml
  [project]
  name = "vienna-weather-forecast-kg-pipeline"
  version = "0.0.1"
  description = "RDF-Connect pipeline to ingest Vienna's weather forecast in a knowledge graph."
  requires-python = "==3.13.*"
  dependencies = [
      "rdfc-runner>=0.0.3",
  ]
  
  [build-system]
  requires = ["hatchling"]
  build-backend = "hatchling.build"
  
  [tool.hatch.build.targets.wheel]
  packages = ["resources"]
  
  [tool.hatch.envs.default]
  type = "virtual"
  path = ".venv"
  system-packages = false
  installer = "uv"
  env-vars = { PYTHONPATH = "src" }
  ```
- [ ] Add an instance of your processor to the pipeline
  - Install your built processor locally
    ```bash
    uv add ../processor/dist/rdfc_translation_processor-0.0.1.tar.gz
    ```
  - Import the semantic definition of your processor in `pipeline.ttl` using `owl:imports`
    ```turtle
    ### Import runners and processors
    <> owl:imports <./.venv/lib/python3.13/site-packages/rdfc_translation_processor/processor.ttl>.
    ```
  - Define input and output channels for your processor
    ```turtle
    ### Define the channels
    <translated> a rdfc:Reader, rdfc:Writer.
    ```
  - Create an instance of your processor and configure it to read from the output channel of the RML mapper and write to the new output channel
- [ ] Update the input channel of the SHACL validator to read from the output channel of your processor
- [ ] Add `rdfc:PyRunner` to the pipeline and attach your processor that needs to be run in Python
  - Import its semantic definition
    ```turtle
    ### Import runners and processors
    <> owl:imports <./.venv/lib/python3.13/site-packages/rdfc_runner/index.ttl>.
    ```
  - Link it to the pipeline
    ```turtle
    ### Define the pipeline
    <> a rdfc:Pipeline;
       rdfc:consistsOf [
           rdfc:instantiates rdfc:NodeRunner;
           rdfc:processor <fetcher>, <logger>, <mappingReader>, <validator>, <reporter>, <sdsifier>, <ingester>;
       ], [
           rdfc:instantiaties rdfc:JvmRunner;
           rdfc:processor <mapper>;
       ], [
           rdfc:instantiates rdfc:PyRunner;
           rdfc:processor <translator>;
       ].
    ```

✅ Complete solution available in **`task-7` branch**  

🎉 You have now completed **Part 2**! The full pipeline now **translates German literals to English** before validation and ingestion into Virtuoso. Run the pipeline with:  

```bash
npx rdfc pipeline.ttl
# or with debug logs:
LOG_LEVEL=debug npx rdfc pipeline.ttl
```

Query Virtuoso and confirm the translated literals are present.
