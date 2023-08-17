---
layout: home
title: Layered Diffusion
permalink: /collage_diffusion/
---

**Vishnu Sarukkai**, Linden Li, Arden Ma, Chris Ré, and Kayvon Fatahalian

[Paper](https://arxiv.org/abs/2303.00262) [Code](https://github.com/VSAnimator/collage-diffusion) [Demo](collagediffusion.stanford.edu)

Diffusion models produce beautiful images, but finding just the right prompt and seed to generate the image you want can be a painstakingly long process. 
This becomes more and more of a problem when trying to generate complex scenes, with many objects in a specific arrangement.
When you finally have an image you mostly like, it still isn't easy to tweak a single object without changing the rest of the image! 

*Collage Diffusion* extends typical diffusion controls--using text prompts, [personalizing]() generation, using [ControlNet]() to preserve edges or pose--to work on individual objects in a scene. **With *Collage Diffusion*, users can create complex scenes in minutes instead of prompt engineering for hours.**

Users specify the image they want to generate in three simple steps: 1) upload (or generate) images of each object you want in the scene, with text descriptions 2) drag them to where they belong in the scene, and 3) write a text prompt corresponding to the scene. We call the image of an object paired with its text a *layer*. Given a sequence of layers, *Collage Diffusion* generates an image that preserves the arrangement and appearance of the individual objects while making the objects "fit together" in the same image. 

[INSERT TEASER]



Text-conditioned diffusion models are incredibly powerful, enabling users to generate high-quality images from a simple text prompt. 
Diffusion models have the potential to enhance artistic pipelines, generate architectural renderings, let people virtually try-on clothes, and more.

There’s just one catch…diffusion models often generate images that are completely different from the images that the user has in mind! In particular, diffusion models struggle to match user intent in scenes that contain several objects. We aim to address the underlying issue that a text prompt simply is not expressive enough to convey compositional intent. 

## Convey Intent with Text-Image Layers

What if there were an expressive, easy-to-create way to communicate compositional intent that is already common in artistic workflows?

Enter layers! Per-object layers are commonly used in image compositing applications such as Photoshop. Layers provide rich information, enabling controllable, editable image generation. 

We define a layer as:
1. A text prompt, paired with
2. An RGBA image containing visual content corresponding to the text prompt

In our work, users communicate compositional intent by providing a full-image prompt, paired with a sequence of layers. We call this combination of layers and prompt a *collage*. For instance, consider this layered representation of a bento box:

<img src="collage_diffusion/bento_def.pdf" alt="drawing" width="800"/>

With a layered representation, we can control a variety of characteristics of diffusion-generated images on a per-object basis, including spatial layout, visual appearance, and edge maps. In addition, we also gain fine-grained control over image harmonization—how strictly we respect the specification of each individual object when generating a cohesive image. 

## Goal: Harmonize, but Preserve

We aim to maximize the following two objectives:
1. Harmonization: the objects in each layer are combined to produce a coherent, self-consistent image
2. Fidelity: we preserve key attributes of the input layers. These may include:
    1. Spatial fidelity: we preserve the spatial arrangement of the objects described by different layers (the sushi is in the right half of the bento box, the ginger in the top left, etc. )
    2. Appearance fidelity: we preserve key visual characteristics of the objects in the input layers (the sushi is salmon sushi that looks like the sushi in the input layers, the ginger is sliced ginger, not whole ginger, etc.)
    3. Structural fidelity: we preserve aspects of the image structure of the input layer: edge maps, pose, etc. (preserve the orientation of the sushi in the input layers) 

Note that *there is an inherent tradeoff between harmonization and fidelity*: to make objects fit together in a coherent image, we have to alter them from the initial layered representation. First, we introduce algorithmic components to shift the pareto curve of the harmonization-fidelity tradeoff. Next, we introduce a mechanism to enable user control of the harmonization-fidelity tradeoff on a per-layer basis. 

## Maintaining Layer Fidelity in the Harmonization Process

One approach to generating a harmonized image from layered input involves constructing an initial composite image from the layers, and then applying diffusion-based image harmonization. [SDEdit](https://arxiv.org/abs/2108.01073) is widely-used in the diffusion community for both harmonization and generating image variations, adding noise to a starting image, then leveraging a learned diffusion model to remove the added noise. The added noise serves to remove details from the starting image while preserving its high-level structure. However, the SDEdit algorithm does not maintain fidelity to any of the aspects of the input layers that we might like to preserve, starting with the spatial arrangement of the layers themselves:

<img src="collage_diffusion/Step1.pdf" alt="drawing" width="800"/>

By leveraging the rich information contained in layer input, we can improve the fidelity of diffusion-generated images along several axes. First, we focus on ensuring that the generated image matches the desired text-region layout (for each layer, the object described by the layer text is generated in the corresponding location). We leverage the alpha channels of the layer images to obtain per-layer masks, and extend the Paint-By-Words mechanism from [eDiffI](https://research.nvidia.com/labs/dir/eDiff-I/) to manipulate U-Net cross attention layers to ensure that objects corresponding to each layer text string are generated in the respective locations. 

<img src="collage_diffusion/Step2.pdf" alt="drawing" width="800"/>

Now, the sushi, ginger, edamame, and rice are generated in the appropriate components of the box, but the sushi and ginger don't quite look right--the visual content in the input layers has not been preserved.

If there is a particular aspect of the layer’s appearance that we would like to preserve but we cannot capture in text, we aim to preserve that by building upon [Textual Inversion](https://textual-inversion.github.io/), an algorithm for personalizing the images generated by diffusion models. We extend the personalization mechanism from Textual Inversion to work with just a single input image, and apply inversion to the image layers individually. 

<img src="collage_diffusion/Step3.pdf" alt="drawing" width="800"/>

Now, we have successfully generated an image that has salmon sushi and sliced sushi ginger, similar to the input layers. 

When desired, we also have the capacity to preserve structural components of the input layers such as edge maps, pose, etc. [ControlNet](https://github.com/lllyasviel/ControlNet) introduced the capacity to control full-image structure via a variety of conditioning signals: edge maps, human pose, etc. This capacity is enabled by an auxiliary network that modifies the outputs of cross-attention layers throughout the diffusion U-Net architecture. Earlier, we spatially restricted the influence of individual words on regions of the generated image by multiplying computed cross-attention weights with per-layer masks. We can use a similar mechanism to scale the effect of ControlNet conditioning on individual layers, multiplying the outputs of the auxiliary network by per-layer masks and weights. 

<img src="collage_diffusion/Step4.pdf" alt="drawing" width="800"/>

Here, we have maintained the edge maps from the initial sequence in the generated image. While this effect may not be optimal for the bento box, there are plenty of situations where artists may want to preserve key components of layer structure. 

## Controlling the Harmonization-Fidelity Tradeoff

While we have introduced various methods of improving layer fidelity in the harmonization process, there is a tradeoff between the extent to which layer characteristics are preserved and the extent to which the layers are effectively harmonized. We allow users to control this tradeoff on a per-layer basis by allowing them to specify the level of noise injected in the SDEdit process on a per-layer basis. Our paper contains several images that benefit qualitatively from having control over this tradeoff. 

## Editing Generated Images

These control mechanisms significantly improve the controllability of compositional image generation, but what if we still are not satisfied with some part of the generated image? Layers enable iterative edits as well! Leveraging the alpha masks extracted from the input layer images, we can selectively re-generate individual components of an image while keeping the rest constant. Consider the following layered conditioning:

<img src="collage_diffusion/cake_def.pdf" alt="drawing" width="800"/>

The following illustrates the iterative process of generating images from the layers until the artist is satisfied:

<img src="collage_diffusion/Stepwise.pdf" alt="drawing" width="800"/>

In the first row, we like most of the second output image, except for the cake. We re-generate variants of the cake in the second row, keeping other image components constant. Now satisfied with the cake, we try out variants of the window in the third row. Note that in each row, noise is only added to the parts of the input layers that we would like to alter. 

## Layers Enable Per-Object Control

Diffusion models have made significant strides in enabling controllable image generation: text-conditional generation for ease of expression, few-shot personalization to capture custom visual concepts, and auxiliary networks to preserve key components of image structure. 
These methods are most effective for single-object scenes, but increasingly struggle as generation scales to many objects. 
Layered representations enable us to adapt and extend these mechanisms to many-object scenes. 

## Links

[Paper](https://arxiv.org/abs/2303.00262) [Code](https://github.com/VSAnimator/collage-diffusion) [Demo](collagediffusion.stanford.edu)


If you're trying to get [Midjourney]() to generate a particular set of objects in particular parts of the image, get ready for hours of prompt engineering.