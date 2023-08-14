---
layout: home
title: One simple trick to make sketch-conditioned image diffusion magical
permalink: /SketchComplete/
---

**Vishnu Sarukkai**, Chris Ré, and Kayvon Fatahalian

**[Code]() | [Demo]()**

**"A medieval castle, realistic"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/castle_s.gif) | ![](SketchComplete/castle_o1.gif) | ![](SketchComplete/castle_p.gif) |

Text-conditioned diffusion models such as [Stable Diffusion](https://huggingface.co/CompVis/stable-diffusion) have captured our imaginations by generating compelling images from a simple text prompt. Recent work has focused on adding additional sources of control for image generation: controlling image layout with bounding boxes, controlling shapes with edge maps, etc. Sketching is a natural and instinctive means of expressing artistic ideas, and we focus on sketch-controlled image generation. Existing methods for sketch-controlled image generation include [ControlNet](), [Sketch-Guided Diffusion](https://sketch-guided-diffusion.github.io), and [DiffSketching](https://arxiv.org/abs/2305.18812). 

While existing sketch-to-image methods show promising results, they have one key flaw: they only work on completed sketches! However, **a typical sketching workflow is an iterative process!** Artists progressively add or remove lines, sometimes constructing basic structures before diving into finer details, at other times focusing on one region of an image before moving on to another. Therefore, **we need sketch-to-image functionality at intermediate stages of the sketching process**.

In *SketchComplete*, we introduce a ControlNet model that generates images conditioned on *partial sketches*. With this ControlNet, *SketchComplete* 1) generates images corresponding to a sketch at various stages of the sketching process, and 2) leverages these images to generate suggested lines that can help guide the artistic process. 

## Existing methods don't work with partial sketches

Prior work is trained on paired datasets of images and completed sketches. When attempting to generate an image from a partial sketch, these methods treat the sketch as completed, so the whitespace in the rest of the sketch is treated as an indicator that the image should not have content that would typically correspond to a stroke in the input sketch. 

With the prompt "a photorealistic house": 

| **Input Sketch** | **Generated Image 1** | **Generated Image 2** |
| :--: | :--: | :--: |
| ![](SketchComplete/house_sketch.png) | ![](SketchComplete/controlnet_3.png) | ![](SketchComplete/controlnet_2.png) |

## Make partial sketches by randomly deleting lines

[Photo-Sketch](https://mtli.github.io/sketch/) is the largest existing dataset of text-captioned images paired with sketches at partial stages of completion. However, this dataset is 1) restricted to sketches of only 1000 images (we would like a larger dataset), 2) the images are all of outdoor scenes (lacking diversity for general text-conditioned generation), and 3) are constructed by tracing over an existing image (imposing a ordering of strokes that may not correspond to the sketching process of many artists). 

Therefore, we programmatically construct our own dataset of captioned images paired with partial sketches. We accomplish this by 1) converting an image to a rasterized edge map with [HED](), 2) [vectorizing]() the edge map into a collection of strokes, and 3) *randomly deleting a fraction of the strokes*. By deleting strokes in arbitrary order, we enable image generation conditional on strokes drawn in any order as well, accomodating various styles of sketching. 

| **Input Image** | **HED Image** | **Partial Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/20000_0.jpg) | ![](SketchComplete/20000.jpg) | ![](SketchComplete/20000_0.png) |
| ![](SketchComplete/20010_0.jpg) | ![](SketchComplete/20010.jpg) | ![](SketchComplete/20010_0.png) |
| ![](SketchComplete/20020_0.jpg) | ![](SketchComplete/20020.jpg) | ![](SketchComplete/20020_0.png) |

We construct our paired dataset using 45000 images from [LAION Art](), and we train a [ControlNet]() model to condition [Stable Diffusion 1.5]() on the image-sketch pairs. The trained model takes a text caption and a partial sketch as inputs, and outputs generated images corresponding to a potential completion of the sketch. 

## Visualize my drawing so far

When an artist isn't quite sure how they'd like to draw a part of an image, we can generate a variety of visual completions given the lines drawn so far. Here, the artist isn't quite sure how they'd like to draw the handle of the mug, so we generate three images that are all valid completions of the initial sketch, showing varying handles paired with the same outline of the body:

![](SketchComplete/SketchComplete/SketchComplete.001.jpeg)

## Help me draw

With these generated images, *SketchComplete* can provide suggestions on potential lines to draw: we generate potential completions of the existing drawing by running HED on the generated images, then averaging these 'completed sketches' to get a 'guiding sketch': 

![](SketchComplete/SketchComplete/SketchComplete.002.jpeg)

## Putting it all together

Sketching is fundamentally an iterative process, where an artists moves across regions of the image and builds details from coarse to fine. *SketchComplete* integrates into this process by generating images corresponding to each stage of the sketch, and the artist can use the 'guiding sketch' to assist them in drawing future lines. Here are a few examples of the iterative process:

**"A ceramic mug"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/mug_s.gif) | ![](SketchComplete/mug_o1.gif) | ![](SketchComplete/mug_p.gif) |

**"A hobbit house with a mailbox"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/hobbit_s.gif) | ![](SketchComplete/hobbit_o1.gif) | ![](SketchComplete/hobbit_p.gif) |

**"A lighthouse at the edge of the ocean"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/light_s.gif) | ![](SketchComplete/light_o1.gif) | ![](SketchComplete/light_p.gif) |

**"A row of brown shoes"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/shoe_s.gif) | ![](SketchComplete/shoe_o1.gif) | ![](SketchComplete/shoe_p.gif) |

## Change the style

The image caption and underlying diffusion backbone can significantly influence both the image visualizations and suggested lines. As with other text-controlled diffusion applications, we can modify the style or content of the generated images through prompting. In the following drawings, we control the style of visualizations for a sports car by changing a single word:

**"A sports car, realistic"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/car_s.gif) | ![](SketchComplete/car_v1_o1.gif) | ![](SketchComplete/car_v1_p.gif) |

**"A sports car, cartoon"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/car_s.gif) | ![](SketchComplete/car_v2_o1.gif) | ![](SketchComplete/car_v2_p.gif) |

**"A sports car, cel shaded"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/car_s.gif) | ![](SketchComplete/car_v3_o1.gif) | ![](SketchComplete/car_v3_p.gif) |

**"A sports car, rusted"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/car_s.gif) | ![](SketchComplete/car_v4_o1.gif) | ![](SketchComplete/car_v4_p.gif) |

## Change the backbone

We have previously seen that a ControlNet trained on one backbone (ex. Stable Diffusion 1.5) still works on fine-tuned versions of that backbone. This property also holds for our partial-sketch ControlNet model, enabling *SketchComplete* to generate suggestions from models fine-tuned for particular domains. For instance, we can use [Ghibli Diffusion]() to generate Ghibli-style characters:

**"A young boy"**

| **Input Sketch** | **Generated Image** | **Guiding Sketch**
|:--:| :--: | :--: |
| ![](SketchComplete/ghibli_s.gif) | ![](SketchComplete/ghibli_o1.gif) | ![](SketchComplete/ghibli_p.gif) |

## Try it out!

In *SketchComplete*, we integrate these capabilities together with the drawing process. At any point in making a sketch, the artist can generate visualizations and stroke modifications, and they can also refine the style of the prompt as they hone in on what they would like to draw. 

**[Code]() | [Demo]()**