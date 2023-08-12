---
layout: home
title: Masterstroke
permalink: /masterstroke/
---

**Vishnu Sarukkai**, Chris Ré, and Kayvon Fatahalian

[Code]() [Demo]()

Sketching stands as a natural and instinctive means of expressing artistic ideas, allowing creators to communicate their intent through lines and strokes. This method involves an iterative process where artists progressively add or remove lines, often constructing basic structures before diving into finer details. This flexible approach captures the essence of artistic exploration.

In this context of creative evolution, a novel frontier emerges that fuses human skill with technological innovation. This is where *Masterstroke* comes into play. *Masterstroke* is a diffusion-powered sketching assistant that 'completes' partial sketches, visualizes full-color images corresponding to the sketches, and suggests new lines for the artist to potentially draw. Integrated into the traditional sketching process, *Masterstroke* encourages artists to freely experiment with sketching ideas and swiftly realize their creative concepts. 

| **Inputs** | **Inputs** | **Inputs** |
|:--:| :--: | :--: |
| ![](masterstroke/ghibli_sketch.png) | ![](masterstroke/castle_sketch.png) | ![](masterstroke/shoe_sketch.png) | 
| *Ghibli style a boy* | *A castle on a hill* | *A row of brown shoes* |
| **Outputs** | **Outputs** | **Outputs** |
| ![](masterstroke/ghibli_image.png) | ![](masterstroke/castle_image.png) | ![](masterstroke/shoe_image.png) | 

Our aims with *Masterstroke*:
1. Navigating Uncertainty: Artists often have a clear plan for some parts of a sketch while grappling with uncertainty for others. Masterstroke aims to address this by offering potential 'completions' for existing segments, bridging the gap between imagination and reality.
2. Guidance in Ambiguity: The creative journey can stall in moments of uncertainty. Masterstroke steps in by suggesting lines and concepts, acting as a guide to propel artistic visions forward.
3. Tailored to Your Style: Artistic expression is personal and unique. Masterstroke acknowledges this by allowing artists to control the style of recommendations, respecting individual creativity.

## Conditioning diffusion on partial sketches

In order to condition a diffusion model on partial sketches, we first construct a dataset that extracts partial sketches from captioned images. Leveraging 50000 images from the LAION Art dataset, we employ [HED]() to convert images into sketch-like representations. We then apply []() to transform the sketches into vectorized strokes to represent the lines drawn by an artist. To simulate partial drawings, a deliberate subset of strokes is omitted.

INSERT GRAPHIC WITH EACH STEP (RANDOM FROM TRAIN DATA WITH SOME PORTION DELETED THAT'S NONTRIVIAL)

We train a custom ControlNet to augment [Stable Diffusion 1.5]() with the capability to handle partial drawings. While a ControlNet for full drawings already exists, it isn't suitable for partial sketches. 

## Complete-a-sketch

Given a partial drawing and a corresponding text prompt, *Masterstroke* can directly leverage the trained ControlNet to generate a variety of images corresponding to the partial drawing. Early in the drawing process, the image is relatively underspecified, and we generate a variety of images:

| **Inputs** | **Inputs** |
|:--:| :--: |
| ![](masterstroke/hobbit_sketch.png) | ![](masterstroke/hobbit_sketch.png) |
| **Outputs** | **Outputs** |
| ![](masterstroke/hobbit_image.png) | ![](masterstroke/hobbit_image2.png) |

Later in the drawing process, the high-level structure of the image is relatively constrained by the drawing:

| **Inputs** | **Inputs** |
|:--:| :--: |
| ![](masterstroke/hobbit_sketch_3.png) | ![](masterstroke/hobbit_sketch_3.png) | 
| **Outputs** | **Outputs** |
| ![](masterstroke/hobbit_image_3.png) | ![](masterstroke/hobbit_image2_3.png) |

## Help me draw/Artist's block

When an artist isn't quite sure how they'd like to draw a part of an image, we can generate a variety of visual completions given the lines drawn so far. Here, the artist isn't quite sure how they'd like to draw the handle of the mug, so we generate three images that are all valid completions of the initial sketch, showing varying handles paired with the same outline of the body:

| **Inputs** | **Inputs** | **Inputs** |
|:--:| :--: | :--: |
| ![](masterstroke/mug_sketch.png) | ![](masterstroke/mug_sketch.png) | ![](masterstroke/mug_sketch.png) |
| *A ceramic mug* | *A ceramic mug* | *A ceramic mug* |
| **Outputs** | **Outputs** | **Outputs** |
| ![](masterstroke/mug_image_1.png) | ![](masterstroke/mug_image_2.png) | ![](masterstroke/mug_image_3.png) |

With these generated images, *Masterstroke* can provide suggestions on potential lines to draw: we generate potential completions of the existing drawing by running HED on the generated images, then visualizing these 'completed sketches' on the same canvas as the drawing. 

| Sketch | Suggested lines |
|:--:| :--: |
| ![](masterstroke/mug_sketch.png) | ![](masterstroke/mug_sketch_shadow.png) |

Tracing any of these suggested lines can help the artist construct the shape of a tricky object and assist in the drawing process. 

## Change the style

The image caption and underlying diffusion backbone can significantly influence both the image visualizations and suggested lines. As with other text-controlled diffusion applications, we can modify the style or content of the generated images through prompting. For instance, consider the following drawing, in which we have drawn a circle corresponding to the tire of a sports car:

<img src="masterstroke/car_wheel.png" alt="drawing" width="256"/>

With the simple change of a caption, we can render the completed drawing in various styles. 

"A sports car, realistic"

| Image 1 | Image 2 | Image 3 |
|:--:| :--: | :--: |
| ![](masterstroke/car_realistic_1.png) | ![](masterstroke/car_realistic_2.png) | ![](masterstroke/car_realistic_3.png) |

"A sports car, cartoon"

| Image 1 | Image 2 | Image 3 |
|:--:| :--: | :--: |
| ![](masterstroke/car_cartoon_1.png) | ![](masterstroke/car_cartoon_2.png) | ![](masterstroke/car_cartoon_3.png) |

"A sports car, cel shaded"

| Image 1 | Image 2 | Image 3 |
|:--:| :--: | :--: |
| ![](masterstroke/car_cel_1.png) | ![](masterstroke/car_cel_2.png) | ![](masterstroke/car_cel_3.png) |

"A sports car, rusted"

| Image 1 | Image 2 | Image 3 |
|:--:| :--: | :--: |
| ![](masterstroke/car_rusted_1.png) | ![](masterstroke/car_rusted_2.png) | ![](masterstroke/car_rusted_3.png) |

We have previously seen that a ControlNet trained on one backbone (ex. Stable Diffusion 1.5) can function effectively on fine-tuned versions of that backbone. This property also holds for our partial-sketch ControlNet model, enabling *Masterstroke* to generate suggestions from models fine-tuned for particular domains. For instance, we can use [Animation Diffusion]() to generate images of animated characters:

| ![](masterstroke/bunny_sketch.png) | ![](masterstroke/tree_sketch.png) | ![](masterstroke/princess_sketch.png) | 
|:--:| :--: | :--: |
| *A bunny* | *A tree* | *A princess* |
| ![](masterstroke/bunny_image.png) | ![](masterstroke/tree_image.png) | ![](masterstroke/princess_image.png) | 

## Try it out!

In *Masterstroke*, we integrate these capabilities together with the drawing process. At any point in making a sketch, the artist can generate visualizations and stroke modifications, and they can also refine the style of the prompt as they hone in on what they would like to draw. 

[Code]() [Demo]()