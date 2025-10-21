## ⭐ Star it — so I know it’s worth to keep improving it.

## CoreML stable diffusion image generation example app

## [Swift package source](https://github.com/swiftuiux/coreml-stable-diffusion-swift)
## [Documentation(API)](https://swiftpackageindex.com/swiftuiux/coreml-stable-diffusion-swift/main/documentation/coreml_stable_diffusion_swift)

The example app for running text-to-image or image-to-image models to generate images using [Apple's Core ML Stable Diffusion implementation](https://github.com/apple/ml-stable-diffusion)

![The concept](https://github.com/swiftuiux/coreml-stable-diffusion-swift-example/blob/main/img/img_08.gif)

 ## How to get generated image

### Step 1 
Place at least one of your prepared split_einsum models into the ‘Local Models’ folder. Find the ‘Document’ folder through the interface by tapping on the ‘Local Models’ button. If the folder is empty, then create a folder named ‘models’. Refer to the folders’ hierarchy in the image below for guidance.
The example app supports only ``split_einsum`` models. In terms of performance ``split_einsum`` is the fastest way to get result.
### Step 2
Pick up the model that was placed at the local folder from the list. Click update button if you added a model while app was launched
### Step 3 
Enter a prompt or pick up a picture and press "Generate" (You don't need to prepare image size manually) It might take up to a minute or two to get the result

![The concept](https://github.com/swiftuiux/coreml-stable-diffusion-swift-example/blob/main/img/img_03.png)

## How it works

### Super short
**words → numbers → math → picture → check**

### So in short:
text → (TextEncoder) → numbers
numbers + noise → (U-Net) → hidden image
hidden image → (VAE Decoder) → real image
real image → (SafetyChecker) → safe output

## Basically

1. **Text Encoding**  
   You type `"a red apple"`.  
   - `vocab.json` + `merges.txt` handle **tokenization** → break it into units like `[a] [red] [apple]`.  
   - `TextEncoder.mlmodelc` maps those tokens into **numerical vectors** in latent space.  

2. **The model’s brain (U-Net)**  
   - Starts with **random noise** (a messy canvas).  
   - Step by step, it **removes noise** and **adds structure**, following the instructions from your text (the vectors from the TextEncoder).  
   - After many steps, what was just noise slowly looks like the picture you asked for.  
   - At this stage, the image is **not yet pixels** (red/green/blue dots). Instead, it exists in **latent space** — a compressed mathematical version of the image.  

3. **Hidden space (Latent space)**  
   - Latent space = the **hidden mathematical space** where the U-Net operates.  
   - Instead of dealing with millions of pixels directly, the model works with a **smaller grid of numbers** that still captures the essence of shapes, colors, and structures.  
   - Think of it like a **sketch or blueprint**: not the full detailed image, but enough to reconstruct it later.  
   - That’s why it’s called *latent* (hidden): the image exists there only as math.  
     - **Latent space = where** → (the canvas the painter is working on).  
     - **U-Net = how** → (the painter’s hand shaping the canvas).  

4. **VAE Decoder**  
   - Once the latent image is ready, `VAEDecoder.mlmodelc` converts it into a real picture (**pixels**).  
   - The opposite direction (picture → latent space) is done by `VAEEncoder.mlmodelc`.  

5. **Safety check**  
   - Finally, `SafetyChecker.mlmodelc` looks at the generated image and checks if it follows **safety rules**.  
   - It runs the image through a separate classifier (another neural net) to predict if the image belongs to restricted categories (e.g. nudity, gore, etc.).  
   - If it does, the checker can:  
     - blur the image,  
     - block the image, or  
     - replace it with a placeholder.  


### Typical set of files for a model und the purpose of each file

| Step | File Name                | Description |
|------|--------------------------|-------------|
| 1    | `vocab.json`             | **The dictionary** – Lists all known words and subwords. Like a reference book for breaking sentences into pieces. |
| 2    | `merges.txt`             | **The word builder** – Defines how small text units (tokens) combine. Like turning syllables into words. |
| 3    | `TextEncoder.mlmodelc`   | **The translator** – Converts tokenized text into vectors. Like encoding a sentence into machine language. |
| 4    | `Unet.mlmodelc`          | **The artist’s hand** – Starts with noise and shapes the image in latent space. Like sketching based on instructions. |
| 4a   | `UnetChunk1.mlmodelc`    | *(Split variant)* **– First brushstroke** – Begins image generation, rough shapes. |
| 4b   | `UnetChunk2.mlmodelc`    | *(Split variant)* **– Second brushstroke** – Adds detail and completes the latent image. |
| 5    | `VAEDecoder.mlmodelc`    | **The lens** – Decodes latent image into real pixels. Like turning a blueprint into a full picture. |
| 6    | `SafetyChecker.mlmodelc` | **The security guard** – Scans the output for unsafe content. Blocks or filters inappropriate results. |
| —    | `VAEEncoder.mlmodelc`    | *(Optional – when converting real images)* **– The compressor** – Shrinks an image into latent form. |


## Model set example
[coreml-stable-diffusion-2-base](https://huggingface.co/pcuenq/coreml-stable-diffusion-2-base/blob/main/coreml-stable-diffusion-2-base_split_einsum_compiled.zip )

### Performance

 The speed can be unpredictable. Sometimes a model will suddenly run a lot slower than before. It appears as if Core ML is trying to be smart in how to schedule things, but doesn’t always optimal.
