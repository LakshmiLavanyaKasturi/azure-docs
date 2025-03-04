---
author: eric-urban
ms.service: cognitive-services
ms.topic: include
ms.date: 03/15/2022
ms.author: eur
---

[!INCLUDE [Header](../../common/swift.md)]

[!INCLUDE [Introduction](intro.md)]

## Prerequisites

[!INCLUDE [Prerequisites](../../common/azure-prerequisites.md)]

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=SWIFT&Pillar=Speech&Product=text-to-speech&Page=quickstart&Section=Prerequisites" target="_target">I ran into an issue</a>

## Set up the environment

The Speech SDK for Swift is distributed as a framework bundle. The framework supports both Objective-C and Swift on both iOS and macOS.

The Speech SDK can be used in Xcode projects as a [CocoaPod](https://cocoapods.org/), or downloaded directly [here](https://aka.ms/csspeech/macosbinary) and linked manually. This guide uses a CocoaPod. Install the CocoaPod dependency manager as described in its [installation instructions](https://guides.cocoapods.org/using/getting-started.html).

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=SWIFT&Pillar=Speech&Product=text-to-speech&Page=quickstart&Section=Set-up-the-environment" target="_target">I ran into an issue</a>

## Synthesize to speaker output

Follow these steps to synthesize speech in a macOS application.

1. Clone the [Azure-Samples/cognitive-services-speech-sdk](https://github.com/Azure-Samples/cognitive-services-speech-sdk) repository to get the [Synthesize audio in Swift on macOS using the Speech SDK](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/swift/macos/text-to-speech) sample project. The repository also has iOS samples. 
1. Navigate to the directory of the downloaded sample app (`helloworld`) in a terminal. 
1. Run the command `pod install`. This will generate a `helloworld.xcworkspace` Xcode workspace containing both the sample app and the Speech SDK as a dependency. 
1. Open the `helloworld.xcworkspace` workspace in Xcode.
1. Open the file named `AppDelegate.swift` and locate the `applicationDidFinishLaunching` and `synthesize` methods as shown here.

    ```swift
    import Cocoa
    
    @NSApplicationMain
    class AppDelegate: NSObject, NSApplicationDelegate, NSTextFieldDelegate {
        var textField: NSTextField!
        var synthesisButton: NSButton!
        
        var inputText: String!
        
        var sub: String!
        var region: String!
    
        @IBOutlet weak var window: NSWindow!
    
        func applicationDidFinishLaunching(_ aNotification: Notification) {
            print("loading")
            // load subscription information
            sub = "YourSubscriptionKey"
            region = "YourServiceRegion"
            
            inputText = ""
            
            textField = NSTextField(frame: NSRect(x: 100, y: 200, width: 200, height: 50))
            textField.textColor = NSColor.black
            textField.lineBreakMode = .byWordWrapping
            
            textField.placeholderString = "Type something to synthesize."
            textField.delegate = self
            
            self.window.contentView?.addSubview(textField)
            
            synthesisButton = NSButton(frame: NSRect(x: 100, y: 100, width: 200, height: 30))
            synthesisButton.title = "Synthesize"
            synthesisButton.target = self
            synthesisButton.action = #selector(synthesisButtonClicked)
            self.window.contentView?.addSubview(synthesisButton)
        }
        
        @objc func synthesisButtonClicked() {
            DispatchQueue.global(qos: .userInitiated).async {
                self.synthesize()
            }
        }
        
        func synthesize() {
            var speechConfig: SPXSpeechConfiguration?
            do {
                try speechConfig = SPXSpeechConfiguration(subscription: sub, region: region)
            } catch {
                print("error \(error) happened")
                speechConfig = nil
            }

            speechConfig?.speechSynthesisVoiceName = "en-US-JennyNeural";

            let synthesizer = try! SPXSpeechSynthesizer(speechConfig!)
            let result = try! synthesizer.speakText(inputText)
            if result.reason == SPXResultReason.canceled
            {
                let cancellationDetails = try! SPXSpeechSynthesisCancellationDetails(fromCanceledSynthesisResult: result)
                print("cancelled, error code: \(cancellationDetails.errorCode) detail: \(cancellationDetails.errorDetails!) ")
                return
            }
        }
        
        func controlTextDidChange(_ obj: Notification) {
            let textFiled = obj.object as! NSTextField
            inputText = textFiled.stringValue
        }   
    }
    ```

1. In `AppDelegate.m`, replace `YourSubscriptionKey` with your Speech resource key, and replace `YourServiceRegion` with your Speech resource region.
1. Optionally in `AppDelegate.m`, include a speech synthesis voice name as shown here: 
    ```swift
    speechConfig?.speechSynthesisVoiceName = "en-US-JennyNeural";
    ```
1. To change the speech synthesis language, replace `en-US-JennyNeural` with another [supported voice](~/articles/cognitive-services/speech-service/supported-languages.md#prebuilt-neural-voices). All neural voices are multilingual and fluent in their own language and English. For example, if the input text in English is "I'm excited to try text to speech" and you set `es-ES-ElviraNeural`, the text is spoken in English with a Spanish accent. If the voice does not speak the language of the input text, the Speech service won't output synthesized audio.
1. Make the debug output visible by selecting **View** > **Debug Area** > **Activate Console**.
1. Build and run the example code by selecting **Product** -> **Run** from the menu or selecting the **Play** button.

After you input some text and select the button in the app, you should hear the synthesized audio played.

> [!div class="nextstepaction"]
> <a href="https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=SWIFT&Pillar=Speech&Product=text-to-speech&Page=quickstart&Section=Synthesize-to-speaker-output" target="_target">I ran into an issue</a>

This quickstart uses the `SpeakText` operation to synthesize a short block of text that you enter. You can also get text from files as described in these guides:
- For information about speech synthesis from a file, see [How to synthesize speech](~/articles/cognitive-services/speech-service/how-to-speech-synthesis.md) and [Improve synthesis with Speech Synthesis Markup Language (SSML)](~/articles/cognitive-services/speech-service/speech-synthesis-markup.md).
- For information about batch synthesis, see [Synthesize long-form text to speech](~/articles/cognitive-services/speech-service/long-audio-api.md). 

## Clean up resources

[!INCLUDE [Delete resource](../../common/delete-resource.md)]


