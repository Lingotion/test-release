Here’s an updated version with fixes, improved clarity, and a table of contents:

---

# **Thespeon Unity Integration Guide**

## **Table of Contents**
- [**Thespeon Unity Integration Guide**](#thespeon-unity-integration-guide)
  - [**Table of Contents**](#table-of-contents)
  - [**Step 1: Install the Thespeon Package from Git**](#step-1-install-the-thespeon-package-from-git)
  - [**Step 2: Add Sample Scene**](#step-2-add-sample-scene)
  - [**Step 3: Get Acquainted with the Unity Editor**](#step-3-get-acquainted-with-the-unity-editor)
  - [**Step 4: Synth Request**](#step-4-synth-request)
    - [**Step 4.1: GUI**](#step-41-gui)
    - [**Step 4.2: Unity NPC Object (Production Friendly)**](#step-42-unity-npc-object-production-friendly)

---

## **Step 1: Install the Thespeon Package from Git**

1. Open **Unity**.  
2. Go to **Unity > Package Manager** from the top menu.  
3. Click the **+** (Add) button → **Install package from Git URL...**  
4. Paste the repository URL:  

   ```
   https://github.com/Lingotion/unity-package.git#sdk-ActorPack-feedback
   ```

5. Click **Install**.  

---

## **Step 2: Add Sample Scene**

1. In **Package Manager**, select the installed package.  
2. Expand the **Samples** section (if available).  
3. Click **"Import"** next to the sample scene.  

   - Unity will copy the sample files into:  

     ```
     Assets/Samples/*Your project*/<version>/
     ```

4. Open the scene from:  

   ```
   Project > Assets > Samples > *Your project* > /<version>/ > Test > Scenes > FullChainRunButton.unity
   ```

5. Now that you have the package installed, you can either start with [Experiment with the GUI sample](using-the-demogui-sample.md).
 or start with the most basic synth request.  
   - If you are familiar with our tool and want to leverage the full creative control, both ways of interacting with the tool are described by [UserModelInput](./UserModelInput.md).  
   - Regardless of which method you choose, you may still want to get acquainted with the Lingotion Thespeon Editor to easily keep track of your imported packs and their status.  

---

## **Step 3: Get Acquainted with the Thespeon Info Window

1. Go to **Unity > Window > Lingotion > Lingotion Thespeon Info** from the top menu.  
2. If you have not already downloaded your packs, make sure to download them via the portal. Follow instructions from [Get Started with Web Portal](./get-started-webportal.md).  
3. Click on **"Import Actor Pack"** or **"Import Language Pack"** to import your downloaded packs into your project.  
   - ![Editor Screenshot](./data/editor1.png?raw=true "Editor Screenshot")  

4. Once you have imported your packs, you can view an overview of imported Actor Packs in **"Imported Actors Overview"** and monitor the imported language packs in **"Imported Languages Overview."**  
   - Example showing imported Actor Packs for `"DetynOliver"` and `"EliasGranhammar"`, and an English language pack (while missing the Swedish language pack):  
   - ![Editor Screenshot](./data/editor2.png?raw=true "Editor Screenshot")  

5. Always check if you have imported the necessary language packs for the synth request.  
   - Example after importing the Swedish language pack:  
   - ![Editor Screenshot](./data/editor3.png?raw=true "Editor Screenshot")  

---

## **Step 4: Synth Request**

You can either use the GUI or choose the production-friendly way:

For experimentation you can start with [Get Started with demo GUI](./using-the-demogui-sample.md), otherwise for advanced use and full creative tailored use, follow below:


---
### ** Unity NPC Object **

1. The Lingotion Thespeon is mainly interacted through a component called the Thespeon Engine, from which you can schedule audio generation from an input text of your choice. 
2. To get started, create an empty GameObject and add the SimpleNarrator.cs script below as a component:
```csharp
using System;
using System.Collections.Generic;
using InputHandler;
using Lingotion.Thespeon.ThespeonRunscripts;
using UnityEngine;

[RequireComponent(typeof(ThespeonEngine))]
[RequireComponent(typeof(AudioSource))]
public class SimpleNarrator : MonoBehaviour
{
    private ThespeonEngine eng;
    private AudioSource audioSource;
    private List<float> audioData;
    private AudioClip audioClip;
    UserModelInput input;
    
    void Start()
    {
        eng = GetComponent<ThespeonEngine>();
        eng.jitterSecondsWaitTime = 1.0f;
        // Create output audio buffer
        audioData = new List<float>();
        // Create a streaming audio clip, which makes the Unity audio thread call OnAudioRead whenever it requests more audio.
        audioClip = AudioClip.Create("ThespeonClip", 1024, 1, 44100, true, OnAudioRead);
        audioSource = GetComponent<AudioSource>();
        audioSource.clip = audioClip;
        audioSource.loop = true;
        audioSource.Play();
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            // Create an input segment with a sample text
            UserSegment testSegment = new UserSegment("Hello! This is a sample text, and I hope you are glad to hear my voice.");
            // Replace ActorName with your actor of choice from your imported actor list in the Lingotion Thespeon Info window.
            input = new UserModelInput("ActorName", new List<UserSegment>() { testSegment });
            // Schedule a Thespeon job with the input, and send the OnAudioPacketReceive as a callback for the audio chunks.
            eng.Synthesize(input, OnAudioPacketReceive);
        }
    }

    // Simply add the received data to the audio buffer. 
    void OnAudioPacketReceive(float[] data)
    {
        lock (audioData)
        {
            audioData.AddRange(data);
        }
    }

    // Whenever the Unity audio thread needs data, it calls this function for us to fill the float[] data. 
    void OnAudioRead(float[] data)
    {
        lock (audioData)
        {
            int currentCopyLength = Mathf.Min(data.Length, audioData.Count);
            // take slice of buffer
            audioData.CopyTo(0, data, 0, currentCopyLength);
            audioData.RemoveRange(0, currentCopyLength);
            if (currentCopyLength < data.Length)
            {
                Array.Fill(data, 0f, currentCopyLength, data.Length - currentCopyLength);
            }
        }
    }
}

```
3. See [User Model Input](https://github.com/Lingotion/unity-package/blob/sdk-ActorPack-feedback/Documentation%7E/UserModelInput.md) for alternative ways of creating the input to the model, but the below workflow is indifferent for how you create your input.
4. Look at the [[Lingotion_Thespeon|API documentation]] for more full documentation.