# Application step 03 - Create your own applications.
  
Let's take a look at general service description of Speech to text. This step covers console application under Windows 10.

https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/get-started https://github.com/Azure-Samples/cognitive-services-speech-sdk - list of examples.

Lets take a look at the demo description of the workshop and GitHub repository. https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/get-started

Get console application from this GitHub repository https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnetcore/translate-speech-to-text

Replace values with northeurope region and from Azure CLI output.

	var config = SpeechTranslationConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
Run(F5) console application, fix bugs, and observe results. Update language configuration to the needed source.

Now you need to send "Speech to text" results to your translator function.
