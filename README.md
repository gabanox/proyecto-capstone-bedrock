
# Configurar agente de Amazon Bedrock, base de conocimiento y grupo de acciones con Streamlit

## Introducción
Esta guía detalla el proceso de configuración de un agente de Amazon Bedrock en AWS, que incluirá la configuración de buckets de S3, una base de conocimiento, un grupo de acciones y una función Lambda. Usaremos el framework Streamlit para la interfaz de usuario. El agente está diseñado para crear dinámicamente un portafolio de una empresa de inversión basado en parámetros específicos y tiene capacidad de preguntas y respuestas sobre los informes del Comité Federal de Mercado Abierto (FOMC). Este ejercicio también incluirá un método para enviar correos electrónicos, pero no estará completamente configurado.

## Requisitos previos
- Una cuenta activa de AWS.
- Familiaridad con servicios de AWS como Amazon Bedrock, S3, Lambda y Cloud9.

## Diagrama

![Diagrama](images/diagram.png)

## Configuración y Configuración Inicial

# Configurar agente de Amazon Bedrock, base de conocimiento y grupo de acciones con Streamlit

## Introducción
Esta guía detalla el proceso de configuración de un agente de Amazon Bedrock en AWS, que incluirá la configuración de buckets de S3, una base de conocimiento, un grupo de acciones y una función Lambda. Usaremos el framework Streamlit para la interfaz de usuario. El agente está diseñado para crear dinámicamente un portafolio de una empresa de inversiones basado en parámetros específicos, y tiene la capacidad de preguntas y respuestas sobre los informes del Comité Federal de Mercado Abierto (FOMC). Este ejercicio también incluirá un método para enviar correos electrónicos, pero no estará completamente configurado.

## Requisitos previos
- Una cuenta activa de AWS.
- Familiaridad con servicios de AWS como Amazon Bedrock, S3, Lambda y Cloud9.

## Diagrama

![Diagrama](images/diagram.png)

## Configuración y Configuración Inicial

### Paso 1: Crear Buckets de S3
- Asegúrate de que estás en la región **us-west-2**. Si necesitas otra región, tendrás que actualizar la variable de región `theRegion` en el archivo de código `InvokeAgent.py`.
- **Bucket de Datos del Dominio**: Crea un bucket S3 para almacenar los datos del dominio. Por ejemplo, llama al bucket S3 `knowledgebase-bedrock-agent-{alias}`. Usaremos la configuración predeterminada.

![Creación de Bucket 1](images/bucket_pic_1.png)

![Creación de Bucket 2](images/bucket_pic_2.png)

- A continuación, descargaremos los datos del dominio desde [aquí](https://github.com/build-on-aws/bedrock-agents-streamlit/tree/main/S3docs). En tu computadora, abre la terminal o el símbolo del sistema y ejecuta los siguientes comandos `curl` para descargar los datos:

* Para **Mac**
  ```linux
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230201.pdf --output ~/Documents/fomcminutes20230201.pdf

    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230322.pdf --output ~/Documents/fomcminutes20230322.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230614.pdf --output ~/Documents/fomcminutes20230614.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230726.pdf --output ~/Documents/fomcminutes20230726.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230920.pdf --output ~/Documents/fomcminutes20230920.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20231101.pdf --output ~/Documents/fomcminutes20231101.pdf
  ```
 
* For **Windows**
```windows
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230201.pdf --output %USERPROFILE%\Documents\fomcminutes20230201.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230322.pdf --output %USERPROFILE%\Documents\fomcminutes20230322.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230614.pdf --output %USERPROFILE%\Documents\fomcminutes20230614.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230726.pdf --output %USERPROFILE%\Documents\fomcminutes20230726.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20230920.pdf --output %USERPROFILE%\Documents\fomcminutes20230920.pdf
    
    curl https://raw.githubusercontent.com/build-on-aws/bedrock-agents-streamlit/main/S3docs/fomcminutes20231101.pdf --output %USERPROFILE%\Documents\fomcminutes20231101.pdf
```
- También tienes la opción de descargar los archivos .pdf desde [aquí](https://github.com/build-on-aws/bedrock-agents-streamlit/tree/main/S3docs). Estos archivos se descargarán en tu carpeta **Documents**. Sube estos archivos al bucket S3 `knowledgebase-bedrock-agent-{alias}`. Estos archivos son documentos del Comité Federal de Mercado Abierto que describen las decisiones de política monetaria tomadas en las reuniones de la Junta de la Reserva Federal. Los documentos incluyen discusiones sobre condiciones económicas, directrices de política para el Banco de la Reserva Federal de Nueva York en operaciones de mercado abierto y votos sobre la tasa de fondos federales. Se puede encontrar más información [aquí](https://www.federalreserve.gov/newsevents/pressreleases/monetary20231011a.htm). Una vez subidos, selecciona uno de los documentos para abrirlo y revisar su contenido.

![Datos del dominio del bucket](images/bucket_domain_data.png)

### Paso 2: Configuración de la Base de Conocimiento en el Agente Bedrock

- Antes de configurar la base de conocimiento, necesitaremos conceder acceso a los modelos que serán necesarios para nuestro agente Bedrock. Navega a la consola de Amazon Bedrock, luego en el lado izquierdo de la pantalla, desplázate hacia abajo y selecciona **Acceso a modelos**. A la derecha, selecciona el botón naranja **Administrar acceso a modelos**.

![Acceso a Modelos](images/model_access.png)

- Marca la casilla para los modelos base **Amazon: Titan Embeddings G1 - Text** y **Anthropic: Claude 3 Haiku**. Esto te proporcionará acceso a los modelos requeridos. Luego, desplázate hacia la parte inferior derecha y selecciona **Solicitar acceso a modelos**.

- Después, verifica que el estado de acceso de los modelos esté en verde con **Acceso concedido**.

![Acceso concedido](images/access_granted.png)

- Ahora, crearemos una base de conocimiento seleccionando **Base de conocimiento** en la izquierda, y luego seleccionando el botón naranja **Crear base de conocimiento**.

![Botón de Crear Base de Conocimiento](images/create_kb_btn.png)

- Puedes usar el nombre predeterminado o ingresar uno propio. Luego, selecciona **Siguiente** en la parte inferior derecha de la pantalla.

![KB details](images/kb_details.gif)

- Sincroniza el bucket S3 `knowledgebase-bedrock-agent-{alias}` con esta base de conocimiento.

![Configuración de la Base de Conocimiento](images/KB_setup.png)

- Para el modelo de embeddings, elige **Titan Embeddings G1 - Text v1.2**. Deja las demás opciones como predeterminadas y desplázate hacia abajo para seleccionar **Siguiente**.

![Configuración del Almacén de Vectores](/static/vector_store_config.gif)

- En la siguiente pantalla, revisa tu trabajo y luego selecciona **Crear base de conocimiento**. 
(Crear la base de conocimiento puede tomar unos minutos. Por favor, espera a que termine antes de continuar con el siguiente paso).

![Revisar y Crear Base de Conocimiento](images/review_create_kb.png)

- Cuando la base de conocimiento esté completa, verás un mensaje en verde en la parte superior similar al siguiente:

![Base de Conocimiento Completa](images/kb_complete.png)

### Paso 3: Configuración de la Función Lambda
- Crea una función Lambda (Python 3.12) para el grupo de acciones del agente Bedrock. Llamaremos a esta función Lambda `PortfolioCreator-actions`.

![Crear Función](images/create_function.png)

![Crear Función 2](images/create_function2.png)

- Copia el código Python proporcionado a continuación, o desde el archivo [aquí](https://github.com/build-on-aws/bedrock-agents-streamlit/blob/main/ActionLambda.py), en tu función Lambda.
  
```python
import json

def lambda_handler(event, context):
    print(event)
  
    # Mock data for demonstration purposes
    company_data = [
        #Technology Industry
        {"companyId": 1, "companyName": "TechStashNova Inc.", "industrySector": "Technology", "revenue": 10000, "expenses": 3000, "profit": 7000, "employees": 10},
        {"companyId": 2, "companyName": "QuantumPirateLeap Technologies", "industrySector": "Technology", "revenue": 20000, "expenses": 4000, "profit": 16000, "employees": 10},
        {"companyId": 3, "companyName": "CyberCipherSecure IT", "industrySector": "Technology", "revenue": 30000, "expenses": 5000, "profit": 25000, "employees": 10},
        {"companyId": 4, "companyName": "DigitalMyricalDreams Gaming", "industrySector": "Technology", "revenue": 40000, "expenses": 6000, "profit": 34000, "employees": 10},
        {"companyId": 5, "companyName": "NanoMedNoLand Pharmaceuticals", "industrySector": "Technology", "revenue": 50000, "expenses": 7000, "profit": 43000, "employees": 10},
        {"companyId": 6, "companyName": "RoboSuperBombTech Industries", "industrySector": "Technology", "revenue": 60000, "expenses": 8000, "profit": 52000, "employees": 12},
        {"companyId": 7, "companyName": "FuturePastNet Solutions", "industrySector": "Technology",  "revenue": 60000, "expenses": 9000, "profit": 51000, "employees": 10},
        {"companyId": 8, "companyName": "InnovativeCreativeAI Corp", "industrySector": "Technology", "revenue": 65000, "expenses": 10000, "profit": 55000, "employees": 15},
        {"companyId": 9, "companyName": "EcoLeekoTech Energy", "industrySector": "Technology", "revenue": 70000, "expenses": 11000, "profit": 59000, "employees": 10},
        {"companyId": 10, "companyName": "TechyWealthHealth Systems", "industrySector": "Technology", "revenue": 80000, "expenses": 12000, "profit": 68000, "employees": 10},
    
        #Real Estate Industry
        {"companyId": 11, "companyName": "LuxuryToNiceLiving Real Estate", "industrySector": "Real Estate", "revenue": 90000, "expenses": 13000, "profit": 77000, "employees": 10},
        {"companyId": 12, "companyName": "UrbanTurbanDevelopers Inc.", "industrySector": "Real Estate", "revenue": 100000, "expenses": 14000, "profit": 86000, "employees": 10},
        {"companyId": 13, "companyName": "SkyLowHigh Towers", "industrySector": "Real Estate", "revenue": 110000, "expenses": 15000, "profit": 95000, "employees": 18},
        {"companyId": 14, "companyName": "GreenBrownSpace Properties", "industrySector": "Real Estate", "revenue": 120000, "expenses": 16000, "profit": 104000, "employees": 10},
        {"companyId": 15, "companyName": "ModernFutureHomes Ltd.", "industrySector": "Real Estate", "revenue": 130000, "expenses": 17000, "profit": 113000, "employees": 10},
        {"companyId": 16, "companyName": "CityCountycape Estates", "industrySector": "Real Estate", "revenue": 140000, "expenses": 18000, "profit": 122000, "employees": 10},
        {"companyId": 17, "companyName": "CoastalFocalRealty Group", "industrySector": "Real Estate", "revenue": 150000, "expenses": 19000, "profit": 131000, "employees": 10},
        {"companyId": 18, "companyName": "InnovativeModernLiving Spaces", "industrySector": "Real Estate", "revenue": 160000, "expenses": 20000, "profit": 140000, "employees": 10},
        {"companyId": 19, "companyName": "GlobalRegional Properties Alliance", "industrySector": "Real Estate", "revenue": 170000, "expenses": 21000, "profit": 149000, "employees": 11},
        {"companyId": 20, "companyName": "NextGenPast Residences", "industrySector": "Real Estate", "revenue": 180000, "expenses": 22000, "profit": 158000, "employees": 260}
    ]
    
  
    def get_named_parameter(event, name):
        return next(item for item in event['parameters'] if item['name'] == name)['value']
    
 
    def companyResearch(event):
        companyName = get_named_parameter(event, 'name').lower()
        print("NAME PRINTED: ", companyName)
        
        for company_info in company_data:
            if company_info["companyName"].lower() == companyName:
                return company_info
        return None
    
    def createPortfolio(event, company_data):
        numCompanies = int(get_named_parameter(event, 'numCompanies'))
        industry = get_named_parameter(event, 'industry').lower()

        industry_filtered_companies = [company for company in company_data
                                       if company['industrySector'].lower() == industry]

        sorted_companies = sorted(industry_filtered_companies, key=lambda x: x['profit'], reverse=True)

        top_companies = sorted_companies[:numCompanies]
        return top_companies

 
    def sendEmail(event, company_data):
        emailAddress = get_named_parameter(event, 'emailAddress')
        fomcSummary = get_named_parameter(event, 'fomcSummary')
    
        # Retrieve the portfolio data as a string
        portfolioDataString = get_named_parameter(event, 'portfolio')
    

        # Prepare the email content
        email_subject = "Portfolio Creation Summary and FOMC Search Results"
        #email_body = f"FOMC Search Summary:\n{fomcSummary}\n\nPortfolio Details:\n{json.dumps(portfolioData, indent=4)}"
    
        # Email sending code here (commented out for now)
    
        return "Email sent successfully to {}".format(emailAddress)   
      
      
    result = ''
    response_code = 200
    action_group = event['actionGroup']
    api_path = event['apiPath']
    
    print("api_path: ", api_path )
    
    if api_path == '/companyResearch':
        result = companyResearch(event)
    elif api_path == '/createPortfolio':
        result = createPortfolio(event, company_data)
    elif api_path == '/sendEmail':
        result = sendEmail(event, company_data)
    else:
        response_code = 404
        result = f"Unrecognized api path: {action_group}::{api_path}"
        
    response_body = {
        'application/json': {
            'body': result
        }
    }
        
    action_response = {
        'actionGroup': event['actionGroup'],
        'apiPath': event['apiPath'],
        'httpMethod': event['httpMethod'],
        'httpStatusCode': response_code,
        'responseBody': response_body
    }

    api_response = {'messageVersion': '1.0', 'response': action_response}
    return api_response

```
- Luego, selecciona **Deploy** en la sección de pestañas de la consola de Lambda. Revisa el código proporcionado antes de continuar con el siguiente paso. Verás que estamos utilizando datos simulados para representar varias empresas en las industrias de tecnología e inmobiliaria, junto con funciones que llamaremos más adelante en este taller.

![Despliegue de Lambda](images/lambda_deploy.png)

- A continuación, aplica una política de recursos a Lambda para conceder acceso al agente Bedrock. Para hacer esto, cambia la pestaña superior de **código** a **configuración** y la pestaña lateral a **Permisos**. Luego, desplázate hasta la sección **Declaraciones de políticas basadas en recursos** y haz clic en el botón **Agregar permisos**.

![Configuración de Permisos](images/permissions_config.png)

![Creación de Política de Recursos Lambda](images/lambda_resource_policy_create.png)

- Selecciona ***Servicio de AWS***, luego usa las siguientes configuraciones para configurar la política basada en recursos:

* ***Servicio*** - `Otro`
* ***ID de la Declaración*** - `allow-bedrock-agent`
* ***Principal*** - `bedrock.amazonaws.com`
* ***ARN de la Fuente*** - `arn:aws:bedrock:us-west-2:{account-id}:agent/*` - (Ten en cuenta que AWS recomienda el menor privilegio posible, por lo que solo un agente permitido puede invocar esta función Lambda. Un * al final del ARN otorga acceso a cualquier agente en la cuenta para invocar esta Lambda. Idealmente, no usaríamos esto en un entorno de producción).
* ***Acción*** - `lambda:InvokeFunction`

![Política de Recursos Lambda](images/lambda_resource_policy.png)

- Una vez que tus configuraciones se vean similares a la captura de pantalla anterior, selecciona ***Guardar*** en la parte inferior.

### Paso 4: Configuración del Agente Bedrock y Grupo de Acciones

- Navega a la consola de Bedrock. Ve al menú de la izquierda y, bajo ***Orquestación***, selecciona ***Agentes***. Proporciona un nombre para el agente, como ***PortfolioCreator***, y luego crea el agente.

- La descripción del agente es opcional, y usaremos el nuevo rol de servicio predeterminado. Para el modelo, selecciona **Anthropic: Claude 3 Haiku**. A continuación, proporciona las siguientes instrucciones para el agente:
  
```instruction
Role: You are an investment analyst responsible for creating portfolios, researching companies, summarizing documents, and formatting emails.

Objective: Assist in investment analysis by generating company portfolios, providing research summaries, and facilitating communication through formatted emails.

1. Portfolio Creation:

    Understand the Query: Analyze the user's request to extract key information such as the desired number of companies and industry.
    Generate Portfolio: Based on the criteria from the request, create a portfolio of companies. Use the template provided to format the portfolio.

2. Company Research and Document Summarization:

    Research Companies: For each company in the portfolio, conduct detailed research to gather relevant financial and operational data.
    Summarize Documents: When a document, like the FOMC report, is mentioned, retrieve the document and provide a concise summary.

3. Email Communication:

    Format Email: Using the email template provided, format an email that includes the newly created company portfolio and any summaries of important documents.
    Send Email: Utilize the provided tools to send an email upon request, That includes a summary of provided responses and and portfolios created.
 
```

- Luego, desplázate hacia la parte superior y **Guarda**.

- Las instrucciones para la herramienta de Analista de Inversiones con IA Generativa describen un marco integral diseñado para ayudar en el análisis de inversiones. Esta herramienta se encarga de crear portafolios personalizados de empresas basados en criterios específicos de la industria, realizar investigaciones exhaustivas sobre estas empresas y resumir documentos financieros relevantes. Además, la herramienta formatea y envía correos electrónicos profesionales que contienen los portafolios y resúmenes de documentos. El proceso implica una adaptación continua a los comentarios de los usuarios y el mantenimiento de una comprensión contextual de las solicitudes en curso para garantizar respuestas precisas y eficientes.

- A continuación, agregaremos un grupo de acciones. Desplázate hacia abajo hasta `Grupos de acciones` y luego selecciona ***Agregar***.

- Llama al grupo de acciones `PortfolioCreator-actions`. Configuraremos el `Tipo de grupo de acciones` en ***Definir con esquemas API***. Las `Invocaciones del grupo de acciones` deben configurarse en ***seleccionar una función Lambda existente***. Para la función Lambda, selecciona `PortfolioCreator-actions`.

- Para el `Esquema del grupo de acciones`, elegiremos ***Definir a través del editor de esquemas en línea***. Reemplaza el esquema predeterminado en el editor de **Esquema OpenAPI en línea** con el esquema proporcionado a continuación. También puedes recuperar el esquema del repositorio [aquí](https://github.com/build-on-aws/bedrock-agent-txt2sql/blob/main/schema/athena-schema.json). Después, selecciona ***Agregar***.
`(Este esquema API es necesario para que el agente de Bedrock conozca la estructura del formato y los parámetros necesarios para que el grupo de acciones interactúe con la función Lambda.)`

```schema
{
  "openapi": "3.0.1",
  "info": {
    "title": "PortfolioCreatorAssistant API",
    "description": "API for creating a company portfolio, search company data, and send summarized emails",
    "version": "1.0.0"
  },
  "paths": {
    "/companyResearch": {
      "post": {
        "description": "Get financial data for a company by name",
        "parameters": [
          {
            "name": "name",
            "in": "query",
            "description": "Name of the company to research",
            "required": true,
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response with company data",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/CompanyData"
                }
              }
            }
          }
        }
      }
    },
    "/createPortfolio": {
      "post": {
        "description": "Create a company portfolio of top profit earners by specifying number of companies and industry",
        "parameters": [
          {
            "name": "numCompanies",
            "in": "query",
            "description": "Number of companies to include in the portfolio",
            "required": true,
            "schema": {
              "type": "integer",
              "format": "int32"
            }
          },
          {
            "name": "industry",
            "in": "query",
            "description": "Industry sector for the portfolio companies",
            "required": true,
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response with generated portfolio",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/Portfolio"
                }
              }
            }
          }
        }
      }
    },
    "/sendEmail": {
      "post": {
        "description": "Send an email with FOMC search summary and created portfolio",
        "parameters": [
          {
            "name": "emailAddress",
            "in": "query",
            "description": "Recipient's email address",
            "required": true,
            "schema": {
              "type": "string",
              "format": "email"
            }
          },
          {
            "name": "fomcSummary",
            "in": "query",
            "description": "Summary of FOMC search results",
            "required": true,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "portfolio",
            "in": "query",
            "description": "Details of the created stock portfolio",
            "required": true,
            "schema": {
              "$ref": "#/components/schemas/Portfolio"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Email sent successfully",
            "content": {
              "text/plain": {
                "schema": {
                  "type": "string",
                  "description": "Confirmation message"
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "CompanyData": {
        "type": "object",
        "description": "Financial data for a single company",
        "properties": {
          "name": {
            "type": "string",
            "description": "Company name"
          },
          "expenses": {
            "type": "string",
            "description": "Annual expenses"
          },
          "revenue": {
            "type": "number",
            "description": "Annual revenue"
          },
          "profit": {
            "type": "number",
            "description": "Annual profit"
          }
        }
      },
      "Portfolio": {
        "type": "object",
        "description": "Stock portfolio with specified number of companies",
        "properties": {
          "companies": {
            "type": "array",
            "items": {
              "$ref": "#/components/schemas/CompanyData"
            },
            "description": "List of companies in the portfolio"
          }
        }
      }
    }
  }
}
```
- Este esquema API define tres endpoints principales: `/companyResearch`, `/createPortfolio` y `/sendEmail`, detallando cómo interactuar con la API, los parámetros requeridos y las respuestas esperadas.

- Ahora, necesitamos proporcionar al agente de Bedrock un prompt que incluya ejemplos de una respuesta formateada para un portafolio de una empresa de inversión y un correo electrónico. Al crear un agente, se configura inicialmente con cuatro plantillas de prompts fundamentales para ***pre-procesamiento***, ***orquestación***, ***generación de respuestas de la base de conocimiento*** y ***post-procesamiento***. Estos prompts guían cómo interactúa el agente con el modelo base en las diversas etapas del proceso. Estas plantillas son cruciales para procesar las entradas del usuario, orquestar el flujo entre el modelo base, los grupos de acciones y las bases de conocimiento, así como para formatear las respuestas enviadas a los usuarios. Al personalizar estas plantillas e incorporar prompts avanzados o ejemplos de few-shot, puedes mejorar significativamente la precisión y el rendimiento del agente en el manejo de tareas específicas. Puedes encontrar más información sobre prompts avanzados para un agente [aquí](https://docs.aws.amazon.com/bedrock/latest/userguide/advanced-prompts.html). Además, existe la opción de usar una [función Lambda personalizada de parser](https://docs.aws.amazon.com/bedrock/latest/userguide/lambda-parser.html) para un formateo más detallado.

- A continuación, desplázate hacia abajo hasta **Prompts avanzados** y selecciona **Editar**.

![Botón de Prompts Avanzados](images/advance_prompt_btn.png)

- Selecciona la pestaña **Orquestación**. Activa el botón de radio **Anular plantillas de orquestación predeterminadas**. Asegúrate de que **Activar plantilla de orquestación** también esté habilitado.

- En el ***Editor de plantillas de prompts***, desplázate hacia abajo hasta las líneas 22-23, luego copia/pega el siguiente ejemplo de portafolio y formato de correo electrónico:
  
```sql
Here is an example of a company portfolio.  

<portfolio_example>

Here is a portfolio of the top 3 real estate companies:

  1. NextGenPast Residences with revenue of $180,000, expenses of $22,000 and profit of $158,000 employing 260 people. 
  
  2. GlobalRegional Properties Alliance with revenue of $170,000, expenses of $21,000 and profit of $149,000 employing 11 people.
  
  3. InnovativeModernLiving Spaces with revenue of $160,000, expenses of $20,000 and profit of $140,000 employing 10 people.

</portfolio_example>

Here is an example of an email formatted. 

<email_format>

Company Portfolio:

  1. NextGenPast Residences with revenue of $180,000, expenses of $22,000 and profit of $158,000 employing 260 people. 
  
  2. GlobalRegional Properties Alliance with revenue of $170,000, expenses of $21,000 and profit of $149,000 employing 11 people.
  
  3. InnovativeModernLiving Spaces with revenue of $160,000, expenses of $20,000 and profit of $140,000 employing 10 people.  


FOMC Report:

  Participants noted that recent indicators pointed to modest growth in spending and production. Nonetheless, job gains had been robust in recent months, and the unemployment rate remained low. Inflation had eased somewhat but remained elevated.
   
  Participants recognized that Russia’s war against Ukraine was causing tremendous human and economic hardship and was contributing to elevated global uncertainty. Against this background, participants continued to be highly attentive to inflation risks.
</email_format>
```
- Los resultados deberían verse similares a lo siguiente:

![Configuración de Prompts Avanzados](images/advance_prompt_setup.gif)

- Desplázate hasta la parte inferior y selecciona el botón ***Guardar y salir***.

- Ahora, verifica para confirmar que la ***Orquestación*** en la sección de ***Prompts avanzados*** ha sido anulada.

![Orquestación Anulada en Prompts Avanzados](images/adv_prompt_overridden.png)

### Paso 5: Configurar la Base de Conocimiento con el Agente Bedrock

- Mientras estés en la consola del agente Bedrock, desplázate hacia abajo hasta ***Base de conocimiento*** y selecciona Agregar. Al integrar la base de conocimiento con el agente, deberás proporcionar instrucciones básicas sobre cómo manejar la base de conocimiento. Por ejemplo, usa lo siguiente:
  
  ```text
  Use this knowledge base when a user asks about data, such as economic trends, company financial statements, or the outcomes of the Federal Open Market Committee meetings.
  ```
  
 ![Agregar Base de Conocimiento](images/add_knowledge_base2.png)

- Revisa tu entrada, luego selecciona ***Agregar***.

- Desplázate hacia la parte superior y selecciona ***Preparar*** para que los cambios realizados se actualicen. Luego, selecciona ***Guardar y salir***.

### Paso 6: Crear un alias

- Crea un alias (nueva versión) y elige un nombre a tu gusto. Una vez hecho esto, asegúrate de copiar tu **ID de Alias** y **ID de Agente**. Los necesitarás en el paso 8.

![Crear Alias](images/create_alias.png)

## Paso 7: Prueba de la Configuración
### Prueba de la Base de Conocimiento
- Mientras estés en la consola de Bedrock, selecciona **Base de Conocimiento** bajo la pestaña de Orquestación, luego la base de conocimiento que creaste. Desplázate hacia abajo hasta la sección de Fuente de Datos y asegúrate de seleccionar el botón **Sincronizar**.

![Sincronización de Base de Conocimiento](images/kb_sync.png)

- Verás una interfaz de usuario a la derecha donde necesitarás seleccionar un modelo. Elige el **modelo Anthropic Claude 3 Haiku**, luego selecciona **Aplicar**.

![Prueba de Selección de Modelo](images/select_model_test.png)

- Ahora deberías tener la capacidad de ingresar prompts en la interfaz de usuario proporcionada.
  
![KB prompt](images/kb_prompt.png)

- Test Prompts:
  ```text
  Give me a summary of financial market developments and open market operations in January 2023.
  ```
  ```text
  Can you provide information about inflation or rising prices?
  ```
  ```text
  What can you tell me about the Staff Review of the Economic & Financial Situation?
  ```
### Prueba del Agente Bedrock
- Mientras estés en la consola de Bedrock, selecciona **Agentes** bajo la pestaña de Orquestación, luego el agente que creaste. Deberías poder ingresar prompts en la interfaz de usuario proporcionada para probar tu base de conocimiento y los grupos de acciones desde el agente.

![Prueba del Agente](images/agent_test.png)

- Ejemplos de prompts para la **Base de Conocimiento**:
  ```text
  Give me a summary of financial market developments and open market operations in January 2023
  ```
  ```text
  Tell me the participants view on economic conditions and economic outlook
  ```
  ```text
  Provide any important information I should know about inflation, or rising prices
  ```
  ```text
  Tell me about the Staff Review of the Economic & financial Situation
  ```

- Ejemplos de prompts para los **Grupos de Acciones**:
  
```text
  Create a portfolio with 3 companies in the real estate industry
```
```text
  Create portfolio of 3 companies that are in the technology industry
```
```text
  Create a new investment portfolio of companies
```
```text
  Do company research on TechStashNova Inc.
```

- Ejemplo de prompt para la Base de Conocimiento (KB) y Grupos de Acciones (AG):
  ```text
  Send an email to test@example.com that includes the company portfolio and FOMC summary
  ```
  `(The logic for this method is not implemented to send emails)`  

## Paso 8: Configuración y Ejecución de la Aplicación Streamlit en EC2 (Opcional)
1. **Obtener la plantilla CF para lanzar la aplicación Streamlit**: Descarga la plantilla de CloudFormation desde [aquí](https://github.com/build-on-aws/bedrock-agents-streamlit/blob/main/ec2-streamlit-template.yaml). Esta plantilla se utilizará para desplegar una instancia de EC2 que tiene el código de Streamlit para ejecutar la interfaz de usuario. ***Ten en cuenta que el rango CIDR y la VPC utilizados en la plantilla de CloudFormation pueden necesitar ser modificados si la subred y la VPC definidas no son aplicables.***

2. **Desplegar la plantilla a través de CloudFormation**:
   - En tu consola de administración, busca y luego ve al servicio CloudFormation.
   - Crea un stack con nuevos recursos (estándar).

   ![Crear stack](images/create_stack.png)

   - Preparar plantilla: Elige una plantilla existente -> Especificar plantilla: Cargar un archivo de plantilla -> sube la plantilla descargada en el paso anterior.

  ![Configurar stack](images/create_stack_config.png)

   - A continuación, proporciona un nombre para el stack, como ***ec2-streamlit***. Mantén el tipo de instancia en el valor predeterminado de t3.small, luego haz clic en Siguiente.

   ![Detalles del stack](images/stack_details.png)

   - En la pantalla de ***Configurar opciones del stack***, deja todas las configuraciones por defecto, luego haz clic en Siguiente.

   - Desplázate hacia abajo hasta la sección de capacidades y reconoce el mensaje de advertencia antes de enviar.

   - Una vez que el stack esté completo, pasa al siguiente paso.

![Stack completo](images/stack_complete.png)

3. **Editar la aplicación para actualizar los ID del agente**:
   - Navega a la consola de administración de instancias EC2. Bajo la sección de instancias, deberías ver `EC2-Streamlit-App`. Selecciona la casilla junto a ella y luego conéctate a través de `EC2 Instance Connect`.

   ![Conexión EC2](images/ec2_connect.gif)

   - A continuación, usa el siguiente comando para editar el archivo InvokeAgent.py:
     ```bash
     sudo vi app/streamlit_app/InvokeAgent.py
     ```

   - Presiona ***i*** para entrar en modo de edición. Luego, actualiza los valores de ***AGENT ID*** y ***Agent ALIAS ID***.

   ![Edición de archivo](images/file_edit.png)
   
   - Después, presiona `Esc`, luego guarda los cambios en el archivo con el siguiente comando:
     ```bash
     :wq!
     ```   

   - Ahora, inicia la aplicación streamlit:
     ```bash
     streamlit run app/streamlit_app/app.py
     ```
  
   - Deberías ver una URL externa. Copia y pega la URL en un navegador web para iniciar la aplicación streamlit.

![IP Externa](images/external_ip.png)

   - Una vez que la aplicación esté en funcionamiento, por favor prueba algunos de los prompts de ejemplo proporcionados. (En el primer intento, si recibes un error, inténtalo de nuevo).

![Aplicación en Ejecución](images/running_app.png)

   - Opcionalmente, puedes revisar los [eventos de rastreo](https://docs.aws.amazon.com/bedrock/latest/userguide/trace-events.html) en el menú de la izquierda de la pantalla. Estos datos incluirán los rastreos de **Preprocesamiento, Orquestación** y **Postprocesamiento**.

![Eventos de Rastreo](images/trace_events.png)

## Limpieza

Después de completar la configuración y las pruebas del Agente Bedrock y la aplicación Streamlit, sigue estos pasos para limpiar tu entorno AWS y evitar cargos innecesarios:
1. Eliminar Buckets S3:
- Navega a la consola de S3.
- Selecciona los buckets "knowledgebase-bedrock-agent-{alias}" y "artifacts-bedrock-agent-creator-{alias}". Asegúrate de que ambos buckets estén vacíos eliminando los archivos.
- Elige 'Eliminar' y confirma ingresando el nombre del bucket.

2. Eliminar la Función Lambda:
- Ve a la consola de Lambda.
- Selecciona la función "PortfolioCreator-actions".
- Haz clic en 'Eliminar' y confirma la acción.

3. Eliminar el Agente Bedrock:
- En la consola de Bedrock, navega a 'Agentes'.
- Selecciona el agente creado, luego elige 'Eliminar'.

4. Anular el Registro de la Base de Conocimiento en Bedrock:
- Accede a la consola de Bedrock, luego navega a "Base de Conocimiento" bajo la pestaña de Orquestación.
- Selecciona y luego elimina la base de conocimiento creada.

5. Limpiar el Entorno Cloud9:
- Navega a la consola de administración de Cloud9.
- Selecciona el entorno de Cloud9 que creaste, luego elimínalo.
