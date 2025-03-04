---
author: eric-urban
ms.service: cognitive-services
ms.topic: include
ms.date: 02/12/2022
ms.author: eur
---

[!INCLUDE [Header](../../common/javascript.md)]

[!INCLUDE [Introduction](intro.md)]

## Prerequisites

[!INCLUDE [Prerequisites](../../common/azure-prerequisites.md)]

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=JAVASCRIPT&Pillar=Speech&Product=speech-to-text&Page=quickstart&Section=Prerequisites" target="_target">I ran into an issue</a>

## Set up the environment

Before you can do anything, you need to install the Speech SDK for JavaScript. If you just want the package name to install, run `npm install microsoft-cognitiveservices-speech-sdk`. For guided installation instructions, see [Set up the development environment](../../../quickstarts/setup-platform.md?pivots=programming-language-javascript).

Use the following `require` statement to import the SDK:

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
```

For more information on `require`, see the [require documentation](https://nodejs.org/en/knowledge/getting-started/what-is-require/).

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=JAVASCRIPT&Pillar=Speech&Product=speech-to-text&Page=quickstart&Section=Set-up-the-environment" target="_target">I ran into an issue</a>


## Recognize speech from a file 

Follow these steps to create a new console application for speech recognition.

1. Open a command prompt where you want the new project, and create a new file named `SpeechRecognition.js`.
1. Copy the following code into `SpeechRecognition.js`:

    ```javascript
    const fs = require('fs');
    const sdk = require("microsoft-cognitiveservices-speech-sdk");
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    speechConfig.speechRecognitionLanguage = "en-US";
    
    function fromFile() {
        let audioConfig = sdk.AudioConfig.fromWavFileInput(fs.readFileSync("YourAudioFile.wav"));
        let speechRecognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
    
        speechRecognizer.recognizeOnceAsync(result => {
            switch (result.reason) {
                case sdk.ResultReason.RecognizedSpeech:
                    console.log(`RECOGNIZED: Text=${result.text}`);
                    break;
                case sdk.ResultReason.NoMatch:
                    console.log("NOMATCH: Speech could not be recognized.");
                    break;
                case sdk.ResultReason.Canceled:
                    const cancellation = CancellationDetails.fromResult(result);
                    console.log(`CANCELED: Reason=${cancellation.reason}`);
            
                    if (cancellation.reason == sdk.CancellationReason.Error) {
                        console.log(`CANCELED: ErrorCode=${cancellation.ErrorCode}`);
                        console.log(`CANCELED: ErrorDetails=${cancellation.errorDetails}`);
                        console.log("CANCELED: Did you update the key and location/region info?");
                    }
                    break;
            }    
            speechRecognizer.close();
        });
    }
    fromFile();
    ```

1. In `SpeechRecognition.js`, replace `YourSubscriptionKey` with your Speech resource key, and replace `YourServiceRegion` with your Speech resource region.
1. In `SpeechRecognition.js`, replace `YourAudioFile.wav` with your own WAV file. This example only recognizes speech from a WAV file. For For information about other audio formats, see [How to use compressed audio files](~/articles/cognitive-services/speech-service/how-to-use-codec-compressed-audio-input-streams.md). This example supports up to 30 seconds audio.
1. To change the speech recognition language, replace `en-US` with another [supported language](~/articles/cognitive-services/speech-service/supported-languages.md). For example, `es-ES` for Spanish (Spain). The default language is `en-us` if you don't specify a language. For details about how to identify one of multiple languages that might be spoken, see [language identification](~/articles/cognitive-services/speech-service/supported-languages.md). 

Run your new console application to start speech recognition from a file:

```console
node.exe SpeechRecognition.js
```

The speech from the audio file should be output as text: 

```console
RECOGNIZED: Text=I'm excited to try speech to text.
```

This example uses the `recognizeOnceAsync` operation to transcribe utterances of up to 30 seconds, or until silence is detected. For information about continuous recognition for longer audio, including multi-lingual conversations, see [How to recognize speech](~/articles/cognitive-services/speech-service/how-to-recognize-speech.md).

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=JAVASCRIPT&Pillar=Speech&Product=speech-to-text&Page=quickstart&Section=Recognize-speech-from-a-file" target="_target">I ran into an issue</a>

> [!NOTE]
> Recognizing speech from a microphone is not supported in Node.js. It's supported only in a browser-based JavaScript environment. For more information, see the [React sample](https://github.com/Azure-Samples/AzureSpeechReactSample) and the [implementation of speech-to-text from a microphone](https://github.com/Azure-Samples/AzureSpeechReactSample/blob/main/src/App.js#L29) on GitHub. The React sample shows design patterns for the exchange and management of authentication tokens. It also shows the capture of audio from a microphone or file for speech-to-text conversions.

## Clean up resources

[!INCLUDE [Delete resource](../../common/delete-resource.md)]

