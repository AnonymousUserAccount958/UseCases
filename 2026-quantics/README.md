# Quantics  2026


---

## Setup

This guide explains how to deploy the Quantum Low-Code Modeler using Docker and run a QAOA-based workflow.

All required components are provided as Docker containers and started using the supplied `docker-compose` file.

---

## 1. Prepare the Environment

Open the `docker` directory and update the `.env` file:

```
PUBLIC_HOSTNAME=<your-public-ip>
```

Important: Use a publicly reachable IP address. Do not use `localhost`.

---

## 2. Start the Components

Run the following commands from the `docker` directory:

```
docker-compose pull
docker-compose up --build
```

Wait until all containers are running. This may take several minutes.

---

## 3. Open the Modeler

Open the following URL:

http://localhost:4242

You should see the Modeler start screen:

![Modeler Initial](docs/graphics/Bild1.png)

The Modeler is pre-configured with:

- endpoints for the low-code backend  
- OpenTOSCA ecosystem workflow  
- QRM repository  

Open the Configuration menu in the toolbar and enter your OpenAI token.

---

## 4. Select an Algorithm

Open the Algorithm Selection view:

![Algorithm Selection](docs/graphics/Bild2.png)

Read the information provided:

![Notes](docs/graphics/Bild3.png)

Enter the following problem description:

```
I have five locations connected by roads. I want to split the locations into two groups so that as many roads as possible go between the groups.
```

![Problem Input](docs/graphics/Bild4.png)

Continue to the next step.

---

## 5. Inspect the Pattern Graph

Click on "Pattern Graph":

![Pattern Graph](docs/graphics/Bild5.png)

Scroll to review the listed components and information, then return using the Back button.

---

## 6. Select the Template

Click on "Select Template":

![Select Template](docs/graphics/Bild6.png)

Select the QAOA template.

This template is also available as `docs/model.json`.

![Template](docs/graphics/Bild7.png)

---

## 7. Transform the Model

Click on "Transform Model":

![Transform](docs/graphics/Bild8.png)

A validation warning may appear:

![Warning](docs/graphics/Bild9.png)

This warning is expected. QAOA automatically adapts to the number of nodes, and the backend configures measurement of all qubits, so no manual specification is required.

---

## 8. Select and Deploy the Workflow

Select a workflow:

![Workflow](docs/graphics/Bild10.png)

Go to History. The top entry is the most recent transformation.

Click on Deploy:

![Deploy](docs/graphics/Bild11.png)

Deployment may take some time.

---

## 9. Execute the Workflow in Camunda

Open:

http://localhost:8090

Log in using the following credentials:

```
user: demo
password: demo
```

Open the Tasklist, select the process, and enter your IP address:

![Task Run](docs/graphics/Bild12.png)

Click Run.

---

## 10. View the Result

Open the Camunda Cockpit.

The result includes the objective function value. In this example, the value is:

```
5
```
![Final Result](docs/graphics/Bild13.png)
---



## Disclaimer of Warranty
Unless required by applicable law or agreed to in writing, Licensor provides the Work (and each Contributor provides its Contributions) on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE. You are solely responsible for determining the appropriateness of using or redistributing the Work and assume any risks associated with Your exercise of permissions under this License.

## Haftungsausschluss
Dies ist ein Forschungsprototyp. Die Haftung für entgangenen Gewinn, Produktionsausfall, Betriebsunterbrechung, entgangene Nutzungen, Verlust von Daten und Informationen, Finanzierungsaufwendungen sowie sonstige Vermögens- und Folgeschäden ist, außer in Fällen von grober Fahrlässigkeit, Vorsatz und Personenschäden, ausgeschlossen.
