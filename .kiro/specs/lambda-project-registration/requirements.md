# Requirements Document

## Introduction

Este documento define los requisitos para una función AWS Lambda escrita en Python que registra proyectos de equipos participantes. La función Lambda recibe el nombre de un equipo y la descripción de su proyecto, valida que la descripción tenga al menos 20 caracteres y simula el guardado del proyecto en una tabla de DynamoDB asignándole un ID Único.

La función sigue las reglas del equipo: arquitectura Serverless en AWS, implementación en Python usando Únicamente la biblioteca estándar (sin librerías externas) y código simple con comentarios en inglés.

## Glossary

- **Lambda_Handler**: Función AWS Lambda en Python que procesa las solicitudes de registro de proyectos y coordina la validación, generación de ID y simulación de guardado.
- **Validator**: Componente responsable de validar que los datos de entrada cumplen las reglas de negocio.
- **Team_Name**: Nombre del equipo que registra el proyecto, proporcionado en la solicitud de entrada.
- **Description**: Descripción textual del proyecto proporcionada por el equipo en la solicitud de entrada.
- **Project_ID**: Identificador Único asignado a cada proyecto registrado.
- **ID_Generator**: Componente que genera identificadores Únicos para los proyectos usando Únicamente la biblioteca estándar de Python.
- **DynamoDB_Simulator**: Componente que simula el guardado de registros en una tabla de DynamoDB sin realizar llamadas de red reales ni usar librerías externas.
- **Project_Record**: Estructura de datos que contiene el Team_Name, la Description y el Project_ID de un proyecto registrado.

## Requirements

### Requirement 1: Recepción de Datos de Entrada

**User Story:** Como organizador del hackathon, quiero que la función Lambda reciba el nombre del equipo y la descripción del proyecto, para poder registrar los proyectos participantes.

#### Acceptance Criteria

1. WHEN a request event is received, THE Lambda_Handler SHALL extract the Team_Name and Description from the event payload.
2. IF the event payload is a dictionary and does not contain a Team_Name, or the Team_Name is null, empty, or whitespace-only, THEN THE Lambda_Handler SHALL return a 400 response with a message indicating the missing Team_Name.
3. IF the event payload is a dictionary and does not contain a Description, or the Description is null, THEN THE Lambda_Handler SHALL return a 400 response with a message indicating the missing Description.
4. IF the event is null or not a dictionary, THEN THE Lambda_Handler SHALL return a 400 response with a message indicating an invalid request payload.
5. IF both the Team_Name and Description are missing or invalid, THEN THE Lambda_Handler SHALL return a 400 response with the Team_Name error message taking priority.

### Requirement 2: Validación de la Descripción

**User Story:** Como organizador del hackathon, quiero validar que la descripción del proyecto tenga al menos 20 caracteres, para asegurar que los equipos describan su proyecto de forma adecuada.

#### Acceptance Criteria

1. WHEN the Description is received, THE Validator SHALL strip leading and trailing whitespace from the Description and verify that the resulting string contains at least 20 characters.
2. IF the Description contains fewer than 20 characters after whitespace stripping, THEN THE Validator SHALL return a validation error indicating that the minimum length is 20 characters.
3. IF the Description contains exactly 20 characters after whitespace stripping, THEN THE Validator SHALL accept the Description as valid.
4. IF the Description is present but is not a string, THEN THE Validator SHALL return a validation error indicating that the Description must be a text value.

### Requirement 3: Generación de ID Único

**User Story:** Como organizador del hackathon, quiero que cada proyecto registrado tenga un ID Único, para poder identificar y rastrear cada proyecto individualmente.

#### Acceptance Criteria

1. WHEN the input validation passes, THE ID_Generator SHALL generate a Project_ID for the project.
2. THE ID_Generator SHALL generate Project_IDs that are unique across at least 1,000 consecutive Lambda invocations.
3. THE Project_ID SHALL be a string value with at least 8 and at most 128 characters.
4. WHEN the ID_Generator generates 1,000 or more Project_IDs, THE ID_Generator SHALL produce Project_IDs with no duplicate values.

### Requirement 4: Simulación de Guardado en DynamoDB

**User Story:** Como arquitecto Serverless, quiero simular el guardado del proyecto en una tabla de DynamoDB, para probar el flujo de registro sin conectarme a una base de datos real.

#### Acceptance Criteria

1. WHEN the DynamoDB_Simulator receives a Team_Name, a Description, and a generated Project_ID, THE DynamoDB_Simulator SHALL create a Project_Record containing the Team_Name, the Description, and the Project_ID.
2. THE DynamoDB_Simulator SHALL store the Project_Record internally to simulate saving to a DynamoDB table, without making external network calls or requiring external libraries.
3. WHILE simulating the save operation, THE DynamoDB_Simulator SHALL return the saved Project_Record as confirmation.
4. THE DynamoDB_Simulator SHALL ensure that every Project_Record created for valid Team_Name and Description inputs contains the exact Team_Name, the exact Description, and the generated Project_ID.

### Requirement 5: Respuesta de la Función Lambda

**User Story:** Como cliente que invoca la función Lambda, quiero recibir una respuesta clara con el resultado del registro, para saber si el proyecto se guardó correctamente o si ocurrió un error.

#### Acceptance Criteria

1. WHEN the project is successfully registered, THE Lambda_Handler SHALL return a 200 response containing the Project_ID and the saved Project_Record.
2. WHEN a validation error occurs, THE Lambda_Handler SHALL return a 400 response containing the validation error message.
3. IF an unexpected error occurs during execution, THEN THE Lambda_Handler SHALL return a 500 response containing an error message that does not expose internal system details such as stack traces or exception information.
4. WHEN the Lambda_Handler returns a Project_Record, THE Lambda_Handler SHALL return a response serializable to JSON such that parsing the JSON produces a Project_Record with the same Team_Name, the same Description, and the same Project_ID as the original.

### Requirement 6: Uso Exclusivo de la Biblioteca Estándar

**User Story:** Como arquitecto Serverless, quiero que la función Lambda use Únicamente la biblioteca estándar de Python, para mantener el código simple y sin dependencias externas.

#### Acceptance Criteria

1. THE Lambda_Handler, including the Validator, ID_Generator, and DynamoDB_Simulator, SHALL be implemented using only the Python standard library without third-party packages.
2. THE ID_Generator SHALL generate unique Project_IDs using only modules from the Python standard library.
3. THE DynamoDB_Simulator SHALL simulate the save operation using only the Python standard library.
4. WHEN the Lambda is invoked, the Lambda_Handler SHALL execute successfully without requiring any third-party packages to be installed.
