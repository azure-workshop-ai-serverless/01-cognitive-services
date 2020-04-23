# Application step 02 - Cognitive services text translations.

The next step is to use Cognitive Services translation samples and integrate them with existing solutions. You can use advanced scenarios with "Speech to text" and then translation. But let's start with a simple case.
The main goal is to combine two tutorials inside one serverless application.

Text translator documentation(quite good)
https://docs.microsoft.com/en-us/azure/cognitive-services/translator/


1. We will be following a slightly adjusted version of this tutorial.
	https://docs.microsoft.com/en-us/azure/cognitive-services/translator/quickstart-translate?pivots=programming-language-csharp

2. Essentially you need to take source code from tutorial above and add it to the new Azure Function, so you can call translation API from code.

3. Get Translation subscription key from Azure CLI output and set it to global variables along with translator text endpoint. Use <textKey1> use setx.
	
	```
	setx TRANSLATOR_TEXT_SUBSCRIPTION_KEY "432432532532"
	setx TRANSLATOR_TEXT_ENDPOINT "https://api.cognitive.microsofttranslator.com/"
	```


4. Create an Azure Function with HTTP trigger in Visual studio.

5. Add namespaces and Json nuget to project
	using System;
	using System.Net.Http;
	using System.Text;
	using System.Threading.Tasks;
	// Install Newtonsoft.Json with NuGet
	using Newtonsoft.Json;

6. Create classes for the JSON response

```
	public class TranslationResult
	{
	    public DetectedLanguage DetectedLanguage { get; set; }
	    public TextResult SourceText { get; set; }
	    public Translation[] Translations { get; set; }
	}

	public class DetectedLanguage
	{
	    public string Language { get; set; }
	    public float Score { get; set; }
	}

	public class TextResult
	{
	    public string Text { get; set; }
	    public string Script { get; set; }
	}

	public class Translation
	{
	    public string Text { get; set; }
	    public TextResult Transliteration { get; set; }
	    public string To { get; set; }
	    public Alignment Alignment { get; set; }
	    public SentenceLength SentLen { get; set; }
	}

	public class Alignment
	{
	    public string Proj { get; set; }
	}

	public class SentenceLength
	{
	    public int[] SrcSentLen { get; set; }
	    public int[] TransSentLen { get; set; }
	}
```

7. Get subscription information from environment variables
	
	```
	string subscriptionKey = Environment.GetEnvironmentVariable("TRANSLATOR_TEXT_SUBSCRIPTION_KEY");
	string endpoint = Environment.GetEnvironmentVariable("TRANSLATOR_TEXT_ENDPOINT");
	```

8. Add translate request class. And change it to output translated values instead of console app.

```
	static public async Task TranslateTextRequest(string subscriptionKey, string endpoint, string route, string inputText)
	{
		using (var client = new HttpClient())
		using (var request = new HttpRequestMessage())
		{
		request.Method = HttpMethod.Post;
		request.RequestUri = new Uri(endpoint + route);
		request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
		request.Headers.Add("Ocp-Apim-Subscription-Key", subscriptionKey);

		HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
		string result = await response.Content.ReadAsStringAsync();
		TranslationResult[] deserializedOutput = JsonConvert.DeserializeObject<TranslationResult[]>(result);
		foreach (TranslationResult o in deserializedOutput)
		{
		    // Print the detected input language and confidence score.
		    Console.WriteLine("Detected input language: {0}\nConfidence score: {1}\n", o.DetectedLanguage.Language, o.DetectedLanguage.Score);
		    // Iterate over the results and print each translation.
		    foreach (Translation t in o.Translations)
		    {
			Console.WriteLine("Translated to {0}: {1}", t.To, t.Text);
		    }
		}
		}

	}
```

9. Call it from function

```
[FunctionName("Function1")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string textToTranslate = req.Query["text"];
            string subscriptionKey = Environment.GetEnvironmentVariable("TRANSLATOR_TEXT_SUBSCRIPTION_KEY");
            string endpoint = Environment.GetEnvironmentVariable("TRANSLATOR_TEXT_ENDPOINT");
            string route = "/translate?api-version=3.0&to=de&to=it&to=ja&to=th";

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            textToTranslate = textToTranslate ?? data?.name;

            string translationResult = await TranslateTextRequest(subscriptionKey, endpoint, route, textToTranslate);

            string responseMessage = string.IsNullOrEmpty(translationResult)
                ? "This HTTP triggered function executed successfully. Pass a Text in the query string or in the request body for a personalized response."
                : $"Hello, {translationResult}. This HTTP triggered function executed successfully.";

            return new OkObjectResult(responseMessage);
        }
```

10. Fix bugs :).
