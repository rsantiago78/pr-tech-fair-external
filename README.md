# pr-tech-fair-external
Repositorio para documentación, recursos y pasos a seguir para configurar el ambiente de desarrollo para el Hackaton del DDEC 2024.

# Objetivo
Desarollar un chatbot capaz de responder preguntas relacionadas con los datos del Censo de Puerto Rico, utilizando los conjuntos de datos provistos en este repositorio. 

# Prerequisitos
Necesitaras un recurso de Azure OpenAI ya preconfigurado y accesible en una suscripción de Azure.  El equipo de Microsoft ya ha preconfigurado uno y solo necesitaras los API Keys y URLs que te serán provistos. Si quieres explorar como hacerlo en una suscripción personal que poseas, puedes referirte a este enlace: [Provision an Azure OpenAI resource](https://microsoftlearning.github.io/mslearn-openai/Instructions/Exercises/03-prompt-engineering.html#provision-an-azure-openai-resource).

# Preparación del ambiente para desarrollo en .Net
1. Descarga Visual Studio Code: [Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)
1. Installar extension **C# Dev Kit for Visual Studio Code** desde VSCode.
1. Installar el [.Net SDK](https://dotnet.microsoft.com/en-us/download)

# Preparación del projecto para uso librerias de Azure.AI.OpenAI 
1. Crear nueva carpeta para el proyecto.
1. Abrir carpeta para el proyecto en VSCode.
1. Podrás utilizar la plantilla de projecto Web que prefieras. Para ver listado de plantillas disponibles puedes ejecutar:
```
dotnet new list
```

4. Inicializar el tipo de proyecto ejecutando 
``` 
dotnet new <project_type>
```



5. Anadir archivo llamado **NuGet.config** con el siguiente contenido
```
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```
6. Instalar el nuget package de [Azure OpenAI](https://www.nuget.org/packages/Azure.AI.OpenAI) ejecutando 
```
dotnet add package Azure.AI.OpenAI --prerelease
```

En este punto, tu ambiente de desarrollo tiene lo necesario para utilizar las librerías de  [Azure OpenAI](https://www.nuget.org/packages/Azure.AI.OpenAI) para realizar llamadas a un recurso de OpenAI en Azure. 

**Nota importante** : Puedes utilizar el IDE de tu predilección, siempre y cuando apoye desarrollo en .NET y las librerías de Azure OpenAI.

# Codigo de Ejemplo

```csharp
using Azure;
using Azure.AI.OpenAI;

String deploymentName = "pr-tech-fair-gpt-35-turbo";
Uri azureOpenAIResourceUri = new("https://pr-tech-fair-aoai.openai.azure.com/");
AzureKeyCredential azureOpenAIApiKey = new("LLAVE_SERA_PROVISTA_POR_EL_EQUIPO");
OpenAIClient client = new(azureOpenAIResourceUri, azureOpenAIApiKey);


// Configure search service
string searchEndpoint = "https://pr-tech-fair-acs.search.windows.net";
string searchKey = "LLAVE_SERA_PROVISTA_POR_EL_EQUIPO";
string searchIndex = "prtechfaircensodataindex";

AzureSearchChatExtensionConfiguration searchConfig = new()
{
    SearchEndpoint = new Uri(searchEndpoint),
    Authentication = new OnYourDataApiKeyAuthenticationOptions(searchKey),
    IndexName = searchIndex
};  
    
var chatCompletionsOptions = new ChatCompletionsOptions()
{
    DeploymentName = deploymentName, // Use DeploymentName for "model" with non-Azure clients
    AzureExtensionsOptions = new AzureChatExtensionsOptions
    {
        Extensions = { searchConfig }
    },
    Messages =
    {
        // The system message represents instructions or other guidance about how the assistant should behave
        new ChatRequestSystemMessage("You are a chat assitant"),
        // User messages represent current or historical input from the end user    
        new ChatRequestUserMessage("What is the total population of students in Puerto Rico?"),
    }
};

Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);
ChatResponseMessage responseMessage = response.Value.Choices[0].Message;
Console.WriteLine($"[{responseMessage.Role.ToString().ToUpperInvariant()}]: {responseMessage.Content}");
```

# Datasets del Censo de Puerto Rico
Podrás descargar localmente los datos el Censo de Puerto Rico que se encuentran bajo la carpeta [datasets\censopr]( https://github.com/rasantia_microsoft/pr-tech-fair-external/tree/main/datasets/censopr). Estos datos fueron obtenidos de [census.gov](https://data.census.gov/table?g=040XX00US72).

# GPT Prompting Cheat Sheet
Utiliza la siguiente formula cuando escribas tus “prompts” para buscar obtener mejores respuestas.

<img width="314" alt="image" src="/images/317050342-01b812b4-364a-46c7-a971-5398672344fb.png">

**Contexto**: Trasfondo o información relevante para el mensaje.
**Tarea**: La función específica que ChatGPT tiene que ejecutar.
**Ejemplos**: Ejemplos ilustrativos o instrucciones guías para el resultado esperado.
**Persona**: Personaje o estilo de para la respuesta por parte del model. (Ej. Experto en programación, Supervisor, YouTuber, CEO, Tutor)
**Formato**: Estructura del resultado. (Ej. Demuéstrame la contestación en “bullets”, la respuesta en tres cortos parafos)
**Tono**: La emoción, o la actitud en la cual deseamos la respuesta, como puede ser formal, o casual.

Ejemplo donde se utiliza la (1) Persona + (2) Contexto + (3) Tarea + (4) Formato + (5) Tono
- (1)	Imagina que eres un supervisor de una plata para la farmacéutica XYZ localizada en Puerto Rico.
- (2)	La planta tiene más de 5 años sin recibir nuevos productos.
- (3)	Haz un memo formal para los empleados informando una nueva manufactura de producto “FrenchFriesPlus 20 mg” en las facilidades.
- (4)	El memo debe de detallar la importancia del producto, el potencial de cambios de operación, y expresar el apoyo necesario del equipo en el desarrollo del mismo.
- (5)	Asegura mantener un tono profesional y positivo, como indicativo del espíritu de innovación y compromiso que tiene la compañía a sus empleados.

**Extra**:
Modificadores de respuestas.
- “No Yapping”: Para respuestas concisas, y directas al grano.
- “Take a Deep breath”: Sinal a ChatGPT a proveer una respuesta “pensada”
- “Explain step-by-step how you reached the conclusion”: Fomenta a que ChatGPT de una respuesta con contexto de la información que provee.



# Enlaces de Apoyo
-	[Azure OpenAI Service REST API reference](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#chat-completions?azure-portal=true)
-	[Apply prompt engineering with Azure OpenAI Service](https://learn.microsoft.com/en-us/training/modules/apply-prompt-engineering-azure-openai/)
-	[Sample WebApp](https://github.com/microsoft/sample-app-aoai-chatGPT)
-	[Prompt engineering - OpenAI API](https://platform.openai.com/docs/guides/prompt-engineering)
- [Anthropic Prompt Library](https://docs.anthropic.com/claude/prompt-library)

