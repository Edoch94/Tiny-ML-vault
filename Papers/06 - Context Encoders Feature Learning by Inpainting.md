---
tags:
  - paper
---
# Abstract
We present an **unsupervised visual feature learning** algorithm driven by **context-based pixel prediction**. By analogy with <u>auto-encoders</u>, we propose **Context Encoders** – a convolutional neural network trained to **generate the contents** of an arbitrary **image region conditioned on its surroundings**. In order to succeed at this task, context encoders need to both <u>understand the content</u> of the entire image, as well as <u>produce a plausible hypothesis</u> for the missing part(s). When training context encoders, we have experimented with both a standard <u>pixel-wise reconstruction loss</u>, as well as a <u>reconstruction plus an adversarial loss</u>. The <u>latter produces much sharper results</u> because it can better handle multiple modes in the output. We found that a context encoder learns a representation that **captures** not just **appearance** but also the **semantics** of visual structures. We quantitatively demonstrate the effectiveness of our learned features for CNN pre-training on classification, detection, and segmentation tasks. Furthermore, context encoders can be used for **semantic inpainting** tasks, either stand-alone or as initialization for non-parametric methods.

---

