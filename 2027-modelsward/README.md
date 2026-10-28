# From Natural Language to Executable Quantum Low-Code Models — Use Case

This repository contains the prototype and the use case for the paper
*"From Natural Language to Executable Quantum Low-Code Models: A Multi-Step Generation and Validation Pipeline"*.

The prototype extends the quantum low-code modeling platform Qonstruct with a validation-driven pipeline:
an LLM only extracts requirements from a natural-language problem description and generates an
intermediate representation (IR) of the required modeling elements. The low-code model is then constructed
deterministically, validated against the modeling language and the extracted requirements, and iteratively
refined based on the validation feedback.


---

## Prerequisites

- Docker
- Node.js ≥ 20.12 and npm (only to run the API and the worker outside Docker)
- An API key for the Mistral API (the evaluation used `mistral-small-4`; the model name is set in
  `src/semanticScore.ts` and `src/worker.ts`)

---

## Getting started

### 1. Configure the environment

Copy `.env.template` to `.env` in the repository root and enter your Mistral API key:

```bash
cp .env.template .env
```

| Variable | Default | Description |
|----------|---------|-------------|
| `MISTRAL_TOKEN` | — (required) | Mistral API key |
| `DATABASE_URL` | `postgresql://app:app@localhost:5433/app` | PostgreSQL that stores the jobs |
| `REDIS_HOST` / `REDIS_PORT` | `localhost` / `6378` | Redis of the job queue |
| `LEQO_BACKEND_URL` | `http://localhost:8000` | LEQO backend for circuit-level models (optional) |

The API and the worker read `.env` on start-up; variables that are already set in the environment take
precedence.

### Start everything with Docker

From the repository root:

```bash
docker-compose up --build
```

This starts PostgreSQL, Redis, the LEQO backend, the API (`http://localhost:3000`) and the worker, and
applies the database migrations. Continue with the [use case](#use-case-quantum-addition).

---

## Use case: quantum addition

The user wants to add two numbers on a quantum computer and measure the result.

The use case is run from the Quantum low-code modeler at http://localhost:4242. Alternatively, all steps
can be performed via the API, e.g. from the Swagger UI at http://localhost:3000/docs; the example request
bodies there are those of this use case.

### Step 1 — Problem description

Open the Quantum low-code modeler at http://localhost:4242 and click on the **Model Generation** button:

![Model Generation button](pictures/Image1.png)

Enter

> Add two numbers on a quantum computer and measure the result.

select the generation mode **Compact** and click **Submit Prompt**:

![Generate Model with AI dialog](pictures/Image2.png)

### Step 2 — Requirements extraction

The LLM extracts six requirements (N = 6). R1–R4 are explicit, because the description states the
addition, the two numbers and the measurement. R5 and R6 are inferred: the quantum addition operates on
quantum registers, so both classical numbers must be encoded, although the user prescribes no encoding.
The numbers themselves are not stated, so they remain the placeholders `x1` and `x2`.

```
[
  [exist, type=block, value=QuantumArithmeticOperator, priority=explicit],   R1
  [exist, type=number, value=x1, priority=explicit],                         R2
  [exist, type=number, value=x2, priority=explicit],                         R3
  [exist, type=block, value=Measurement, priority=explicit],                 R4
  [exist, type=encoding, value=Basis, priority=inferred],                    R5
  [exist, type=encoding, value=Basis, priority=inferred]                     R6
]
```

### Step 3 — Element & relationship identification

From the description, the checklist and the language definition, the LLM generates the IR. The values of
the two numbers are left empty, because they are not given in the description.

### Step 4 — Parameter completion

The job pauses and asks for the two operands. Click on the **History** button:

![History button](pictures/Image3.png)

In the **Generation** tab, the job has the status `User input required`. Click **Add Required Values**
(use the refresh button in the upper right corner if the status is not yet updated):

![Job waiting for user input](pictures/Image4.png)

Enter `2` for the *First Operand* and `4` for the *Second Operand* and click **Generate Model**:

![Add Required Values dialog](pictures/Image5.png)

The values are inserted into the IR and bound to the placeholders (`x1 = 2`, `x2 = 4`), so R2 and R3 are
only satisfied if the model contains exactly these values. The resulting initial IR:

```
[
  [id: "Num_0", step: "Number", value: 2, inputs: [], outputs: [QArithm_0]],
  [id: "Num_1", step: "Number", value: 4, inputs: [], outputs: [QArithm_0]],
  [id: "QArithm_0", step: "QuantumArithmeticOperator", variant: "Addition",
   inputs: [Num_0, Num_1], outputs: [Msr_0]],
  [id: "Msr_0", step: "Measurement", inputs: [QArithm_0], outputs: []]
]
```

### Steps 5–7 — Model construction, validation and refinement

The initial candidate connects the classical numbers directly to the quantum addition. Metamodel validation
detects that a classical output is connected to a quantum input, and semantic validation confirms the
deficiency: all four explicit requirements are satisfied, but neither inferred encoding requirement is.
The validation report is passed back to the LLM, which revises the IR:

| Iteration | Syntax | Metamodel | S(C) | S_inf(C) | Result |
|-----------|--------|-----------|------|----------|--------|
| Initial candidate | passed | failed | 0.67 | 0.00 | rejected |
| Refinement 1 (one encoder added) | passed | failed | 0.83 | 0.50 | rejected |
| Refinement 2 (second encoder added) | passed | passed | 1.00 | 1.00 | **accepted** |

The accepted IR:

```
[
  [id: "Num_0", step: "Number", value: 2, inputs: [], outputs: [BE_0]],
  [id: "BE_0", step: "BasisEncoder", inputs: [Num_0], outputs: [QArithm_0]],
  [id: "Num_1", step: "Number", value: 4, inputs: [], outputs: [BE_1]],
  [id: "BE_1", step: "BasisEncoder", inputs: [Num_1], outputs: [QArithm_0]],
  [id: "QArithm_0", step: "QuantumArithmeticOperator", variant: "Addition",
   inputs: [BE_0, BE_1], outputs: [Msr_0]],
  [id: "Msr_0", step: "Measurement", inputs: [QArithm_0], outputs: []]
]
```

The number of refinement iterations depends on the LLM output and may differ between runs.

### Step 8 — Result

Click on the **History** button again:

![History button](pictures/Image6.png)

Once the job is `completed` with a semantic score of `100%`, click **Load Model**:

![Completed job](pictures/Image7.png)

The generated low-code model is loaded into the modeler:

![Generated low-code model](pictures/Image8.png)

For a circuit-level model, select the generation mode **Circuit** in Step 1; the validated compact model is
then transformed into a circuit-level model via OpenQASM (requires the LEQO backend).

The complete trace of the run — the checklist with the bound values, every candidate and all validation
results — is available as PDF via the **Report** button in the History, or via the API:

### Step 9 — Transformation and execution

Click on the **Model Transformation** button. Some warnings may appear during the transformation; confirm
each dialog with the blue button:

![Model Transformation button](pictures/Image9.png)

Then click on the **History** button, open the **Execution** tab and click **Execute Circuit**. Again,
confirm the following dialogs with the blue button.

![Execute Circuit](pictures/Image10.png)
![Result](pictures/Image11.png)
