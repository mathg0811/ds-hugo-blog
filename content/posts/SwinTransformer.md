+++
date = '2025-04-23T20:00:00+09:00'  # 작성 날짜
draft = false  # 초안 여부
title = 'ConvNext'  # 글 제목
tags = ['vision transformers', 'machine learning', 'computer vision']  # 태그 목록
categories = ['vision transformers', 'computer vision']  # 카테고리 목록
url = '/posts/SwinTransformer'  # 커스텀 URL 경로
description = 'SwinTransformer'  # 페이지 설명
images = ['/images/huggingface.jpg']  # 대표 이미지 
+++


# Swin Transformer  

Introduced in the 2021 paper, [Swin Transformer: Hierarchical Vision Transformer using Shifted Windows](https://arxiv.org/pdf/2103.14030.pdf), the Swin Transformer architecture optimizes latency and performance by using a shifted window approach, which reduces the number of operations compared to traditional sliding windows. Swin Transformer is considered a **hierarchical backbone** for computer vision.

Unlike traditional Vision Transformers (ViT), which struggle with computational complexity and fixed-scale features, Swin Transformer introduces a **hierarchical design** and **shifted window attention** to efficiently handle multi-scale visual data. By combining the flexibility of Transformers with the scalability of CNNs, it achieves state-of-the-art results across image classification, object detection, and semantic segmentation tasks.

---

**My take:**  
Scalability isn’t optional in vision—it’s the price of admission. Swin Transformer elegantly meets this demand, unlocking Transformers for high-resolution medical imaging, video analysis, and beyond. While Transformers excel at **modeling relationships between features through self-attention**, CNNs have historically dominated **multi-scale feature extraction via hierarchical layers**. The future isn’t CNNs versus Transformers—it’s architectures like Swin that **combine the strengths of both**: leveraging attention’s ability to **dynamically capture feature dependencies** while adopting CNN-like **pyramidal downsampling** for scalable multi-resolution processing. Swin Transformer demonstrates how to **integrate and refine** established design philosophies by deeply understanding both the task (e.g., object detection) and the environment (e.g., GPU memory constraints).

---

## Design Concept
### **Why Scalability Defines Swin’s Success**  
Traditional Transformers excel in NLP by processing fixed-scale tokens (words), but this approach fails in vision. Images demand **multi-scale reasoning**—a distant car, a rotated sign, or a high-resolution texture all require adaptive feature extraction. Swin Transformer addresses this by reimagining attention mechanisms with **scalability at its core**, bridging the gap between CNNs and Transformers.

---

### **The Scalability Crisis in Vision Transformers**  
- **Fixed Tokens ≠ Visual Reality**:  
  ViT processes images as rigid 16×16 patches, forcing a single-scale representation. Real-world vision requires hierarchical features, from pixels to objects.  
- **Quadratic Computational Complexity**:  
  Global self-attention in ViT becomes computationally prohibitive for high-resolution images (e.g., 4K video or medical scans).  

---

### **Swin’s Scalability Solutions**  
#### 1. **Hierarchical Feature Pyramids**  
Swin mimics CNNs by merging patches in deeper layers (4×4 → 8×8 → 16×16), creating multi-scale feature maps. This enables seamless integration with vision techniques like Feature Pyramid Networks (FPN) for object detection.  
#### 2. **Shifted Window Attention**  
Self-attention is computed within **local windows** (e.g., 7×7 patches) that shift in alternating layers. This reduces complexity to linear scaling while enabling cross-window communication.  

---

### **Swin Transformer vs. ViT: Scalability Showdown**  
| Feature                | Swin Transformer (Scalability First)               | Vision Transformer (ViT)                     |
|------------------------|----------------------------------------------------|----------------------------------------------|
| **Scale Handling**      | Multi-stage merging (CNN-like hierarchy)           | Fixed 16×16 patches                          |
| **Attention Scope**     | Local windows + cross-window shifts                | Global across all patches                    |
| **Complexity**          | Linear with image size                             | Quadratic with image size                    |
| **Backbone Flexibility**| Plug-and-play for FPN, U-Net, etc.                 | Requires task-specific adaptations           |
| **Real-World Performance** | 63.1 COCO box AP (V2), 59.9 mIoU on ADE20K | Lower accuracy on dense tasks (+2.7 AP gap)  |

---

**Swin Variants & Related Models**  
- **SwinV2** ([paper](https://arxiv.org/abs/2111.09883)): Improved training stability and support for higher-resolution images (up to 1536×1536) with Res-Post-Norm and scaled cosine attention.  
- **SwinIR** ([paper](https://arxiv.org/abs/2108.10257)): A Swin Transformer-based model tailored for image restoration tasks like super-resolution and denoising.  
- **Swin2SR** ([paper](https://arxiv.org/abs/2209.09345)): An enhanced and lightweight version of SwinIR optimized for real-time image super-resolution.  
- **Swin-UNETR** ([paper](https://arxiv.org/abs/2103.14030)): Combines Swin Transformer with U-Net architecture for 3D medical image segmentation.  
- **CSWin Transformer** ([paper](https://arxiv.org/abs/2107.00652)): Introduces cross-shaped window attention to overcome local window limitations in Swin.

Each of these variants builds upon the hierarchical design and shifted window mechanism of the original Swin Transformer, adapting it to specific vision tasks and improving performance or efficiency accordingly.

---

</Tip>

## Main Highlights
### Shifted windows
In the original ViT, attention is done between each patch and all other patches, which gets computationally intensive. Swin optimizes this process by reducing the normally quadratic complexity ViT into linear complexity (with respect to image size). Swin achieves this using a technique similar to CNN, where patches only attend to other patches in the same window, as opposed to all other patches, and then are gradually merged with neighboring patches. This is what makes Swin a hierarchical model.

![Architecture Diagram of Swin vs Vit, taken from Swin transformer paper](https://huggingface.co/datasets/hf-vision/course-assets/resolve/main/swin_transformer_architecture.png)
_Image taken from Swin Transformer paper_

## Advantages
### Computational efficiency
Swin is more performant than completely patch-based approaches like ViT.
### Large datasets
SwinV2 is one of the first 3B parameter models. As training size goes up, Swin outperforms CNN. The large number of parameters enables increased capacity for learning and more complex representations.

## Overview of PyTorch Implementation of Swin

### Swin Transformer class
1. **Initialize Parameters**. Among various other dropout and normalization parameters, these parameters include:
    - `window_size`: Size of the windows for local self-attention.
    - `ape (bool)`: If True, add absolute position embedding to the patch embedding. 
    - `fused_window_process`: Optional hardware optimization.

2. **Apply Patch Embedding**: Similar to ViT, Images are split into non-overlapping patches and linearly embedded using `Conv2D`.
    
3. **Apply Positional Embeddings**: `SwinTransformer` optionally uses absolute position embeddings (`ape`), added to the patch embeddings. Absolute positional embeddings often help the model learn to use positional information about each patch to make more informed predictions.

4. **Apply Depth Decay**: Depth decay helps with regularization and preventing overfitting. Depth decay usually done by skipping layers during training. In this Swin implementation, **stochastic** depth decay is used, which means the deeper the layer, the higher the chance it might be skipped.
    
4. **Layer Construction**:
    - The model is composed of multiple layers (`BasicLayer`) of `SwinTransformerBlock`s, each downsampling the feature map for hierarchical processing using `PatchMerging`.
    - The dimensionality of features and resolution of feature maps change across layers.
      
7. **Classification Head**: Similar to ViT, it uses an Multi-Layer Perceptron (MLP) head for classification tasks, as defined in `self.head`, as the last step.

```python
class SwinTransformer(nn.Module):
    def __init__(
        self,
        img_size=224,
        patch_size=4,
        in_chans=3,
        num_classes=1000,
        embed_dim=96,
        depths=[2, 2, 6, 2],
        num_heads=[3, 6, 12, 24],
        window_size=7,
        mlp_ratio=4.0,
        qkv_bias=True,
        qk_scale=None,
        drop_rate=0.0,
        attn_drop_rate=0.0,
        drop_path_rate=0.1,
        norm_layer=nn.LayerNorm,
        ape=False,
        patch_norm=True,
        use_checkpoint=False,
        fused_window_process=False,
        **kwargs,
    ):
        super().__init__()

        self.num_classes = num_classes
        self.num_layers = len(depths)
        self.embed_dim = embed_dim
        self.ape = ape
        self.patch_norm = patch_norm
        self.num_features = int(embed_dim * 2 ** (self.num_layers - 1))
        self.mlp_ratio = mlp_ratio

        # split image into non-overlapping patches
        self.patch_embed = PatchEmbed(
            img_size=img_size,
            patch_size=patch_size,
            in_chans=in_chans,
            embed_dim=embed_dim,
            norm_layer=norm_layer if self.patch_norm else None,
        )
        num_patches = self.patch_embed.num_patches
        patches_resolution = self.patch_embed.patches_resolution
        self.patches_resolution = patches_resolution

        # absolute position embedding
        if self.ape:
            self.absolute_pos_embed = nn.Parameter(torch.zeros(1, num_patches, embed_dim))
            trunc_normal_(self.absolute_pos_embed, std=0.02)

        self.pos_drop = nn.Dropout(p=drop_rate)

        # stochastic depth
        dpr = [
            x.item() for x in torch.linspace(0, drop_path_rate, sum(depths))
        ]  # stochastic depth decay rule

        # build layers
        self.layers = nn.ModuleList()
        for i_layer in range(self.num_layers):
            layer = BasicLayer(
                dim=int(embed_dim * 2**i_layer),
                input_resolution=(
                    patches_resolution[0] // (2**i_layer),
                    patches_resolution[1] // (2**i_layer),
                ),
                depth=depths[i_layer],
                num_heads=num_heads[i_layer],
                window_size=window_size,
                mlp_ratio=self.mlp_ratio,
                qkv_bias=qkv_bias,
                qk_scale=qk_scale,
                drop=drop_rate,
                attn_drop=attn_drop_rate,
                drop_path=dpr[sum(depths[:i_layer]) : sum(depths[: i_layer + 1])],
                norm_layer=norm_layer,
                downsample=PatchMerging if (i_layer < self.num_layers - 1) else None,
                use_checkpoint=use_checkpoint,
                fused_window_process=fused_window_process,
            )
            self.layers.append(layer)

        self.norm = norm_layer(self.num_features)
        self.avgpool = nn.AdaptiveAvgPool1d(1)
        self.head = (
            nn.Linear(self.num_features, num_classes)
            if num_classes > 0
            else nn.Identity()
        )

        self.apply(self._init_weights)

    def _init_weights(self, m):
        if isinstance(m, nn.Linear):
            trunc_normal_(m.weight, std=0.02)
            if isinstance(m, nn.Linear) and m.bias is not None:
                nn.init.constant_(m.bias, 0)
        elif isinstance(m, nn.LayerNorm):
            nn.init.constant_(m.bias, 0)
            nn.init.constant_(m.weight, 1.0)

    @torch.jit.ignore
    def no_weight_decay(self):
        return {"absolute_pos_embed"}

    @torch.jit.ignore
    def no_weight_decay_keywords(self):
        return {"relative_position_bias_table"}

    def forward_features(self, x):
        x = self.patch_embed(x)
        if self.ape:
            x = x + self.absolute_pos_embed
        x = self.pos_drop(x)

        for layer in self.layers:
            x = layer(x)

        x = self.norm(x)  # B L C
        x = self.avgpool(x.transpose(1, 2))  # B C 1
        x = torch.flatten(x, 1)
        return x

    def forward(self, x):
        x = self.forward_features(x)
        x = self.head(x)
        return x
```


### Swin Transformer Block
The `SwinTransformerBlock` encapsulates the core operations of the Swin Transformer: local windowed attention and subsequent MLP processing. It plays a key role in enabling the Swin Transformer to efficiently handle large images by focusing on local patches while maintaining the ability to learn global representations.

**Layer Components**:
- **Normalization Layer 1 (`self.norm1`)**: Applied before the attention mechanism.
- **Window Attention (`self.attn`)**: Computes self-attention within local windows.
- **Drop Path (`self.drop_path`)**: Implements stochastic depth for regularization.
- **Normalization Layer 2 (`self.norm2`)**: Applied before the MLP layer.
- **MLP (`mlp`)**: A multi-layer perceptron for processing features post-attention.
- **Attention Mask (`self.register_buffer`)**: The attention mask is used during the self-attention computation to control which elements in the windowed input are allowed to interact (i.e., attend to each other). The shifted window approach helps the model to capture broader contextual information by allowing some cross-window interaction.
#### Swin Transformer Block's Initialization
```python
class SwinTransformerBlock(nn.Module):
    r"""Swin Transformer Block.

    Args:
        dim (int): Number of input channels.
        input_resolution (tuple[int]): Input resulotion.
        num_heads (int): Number of attention heads.
        window_size (int): Window size.
        shift_size (int): Shift size for SW-MSA.
        mlp_ratio (float): Ratio of mlp hidden dim to embedding dim.
        qkv_bias (bool, optional): If True, add a learnable bias to query, key, value. Default: True
        qk_scale (float | None, optional): Override default qk scale of head_dim ** -0.5 if set.
        drop (float, optional): Dropout rate. Default: 0.0
        attn_drop (float, optional): Attention dropout rate. Default: 0.0
        drop_path (float, optional): Stochastic depth rate. Default: 0.0
        act_layer (nn.Module, optional): Activation layer. Default: nn.GELU
        norm_layer (nn.Module, optional): Normalization layer.  Default: nn.LayerNorm
        fused_window_process (bool, optional): If True, use one kernel to fused window shift & window partition for acceleration, similar for the reversed part. Default: False
    """

    def __init__(
        self,
        dim,
        input_resolution,
        num_heads,
        window_size=7,
        shift_size=0,
        mlp_ratio=4.0,
        qkv_bias=True,
        qk_scale=None,
        drop=0.0,
        attn_drop=0.0,
        drop_path=0.0,
        act_layer=nn.GELU,
        norm_layer=nn.LayerNorm,
        fused_window_process=False,
    ):
        super().__init__()
        self.dim = dim
        self.input_resolution = input_resolution
        self.num_heads = num_heads
        self.window_size = window_size
        self.shift_size = shift_size
        self.mlp_ratio = mlp_ratio
        if min(self.input_resolution) <= self.window_size:
            # if window size is larger than input resolution, we don't partition windows
            self.shift_size = 0
            self.window_size = min(self.input_resolution)
        assert 0 <= self.shift_size < self.window_size, "shift_size must in 0-window_size"

        self.norm1 = norm_layer(dim)
        self.attn = WindowAttention(
            dim,
            window_size=to_2tuple(self.window_size),
            num_heads=num_heads,
            qkv_bias=qkv_bias,
            qk_scale=qk_scale,
            attn_drop=attn_drop,
            proj_drop=drop,
        )

        self.drop_path = DropPath(drop_path) if drop_path > 0.0 else nn.Identity()
        self.norm2 = norm_layer(dim)
        mlp_hidden_dim = int(dim * mlp_ratio)
        self.mlp = Mlp(
            in_features=dim,
            hidden_features=mlp_hidden_dim,
            act_layer=act_layer,
            drop=drop,
        )

        if self.shift_size > 0:
            # calculate attention mask for SW-MSA
            H, W = self.input_resolution
            img_mask = torch.zeros((1, H, W, 1))  # 1 H W 1
            h_slices = (
                slice(0, -self.window_size),
                slice(-self.window_size, -self.shift_size),
                slice(-self.shift_size, None),
            )
            w_slices = (
                slice(0, -self.window_size),
                slice(-self.window_size, -self.shift_size),
                slice(-self.shift_size, None),
            )
            cnt = 0
            for h in h_slices:
                for w in w_slices:
                    img_mask[:, h, w, :] = cnt
                    cnt += 1

            mask_windows = window_partition(
                img_mask, self.window_size
            )  # nW, window_size, window_size, 1
            mask_windows = mask_windows.view(-1, self.window_size * self.window_size)
            attn_mask = mask_windows.unsqueeze(1) - mask_windows.unsqueeze(2)
            attn_mask = attn_mask.masked_fill(attn_mask != 0, float(-100.0)).masked_fill(
                attn_mask == 0, float(0.0)
            )
        else:
            attn_mask = None

        self.register_buffer("attn_mask", attn_mask)
        self.fused_window_process = fused_window_process

    ### New cell ###
    def forward(self, x):
        H, W = self.input_resolution
        B, L, C = x.shape
        assert L == H * W, "input feature has wrong size"

        shortcut = x
        x = self.norm1(x)
        x = x.view(B, H, W, C)

        # cyclic shift
        if self.shift_size > 0:
            if not self.fused_window_process:
                shifted_x = torch.roll(
                    x, shifts=(-self.shift_size, -self.shift_size), dims=(1, 2)
                )
                # partition windows
                x_windows = window_partition(
                    shifted_x, self.window_size
                )  # nW*B, window_size, window_size, C
            else:
                x_windows = WindowProcess.apply(
                    x, B, H, W, C, -self.shift_size, self.window_size
                )
        else:
            shifted_x = x
            # partition windows
            x_windows = window_partition(
                shifted_x, self.window_size
            )  # nW*B, window_size, window_size, C

        x_windows = x_windows.view(
            -1, self.window_size * self.window_size, C
        )  # nW*B, window_size*window_size, C

        # W-MSA/SW-MSA
        attn_windows = self.attn(
            x_windows, mask=self.attn_mask
        )  # nW*B, window_size*window_size, C

        # merge windows
        attn_windows = attn_windows.view(-1, self.window_size, self.window_size, C)

        # reverse cyclic shift
        if self.shift_size > 0:
            if not self.fused_window_process:
                shifted_x = window_reverse(
                    attn_windows, self.window_size, H, W
                )  # B H' W' C
                x = torch.roll(
                    shifted_x, shifts=(self.shift_size, self.shift_size), dims=(1, 2)
                )
            else:
                x = WindowProcessReverse.apply(
                    attn_windows, B, H, W, C, self.shift_size, self.window_size
                )
        else:
            shifted_x = window_reverse(attn_windows, self.window_size, H, W)  # B H' W' C
            x = shifted_x
        x = x.view(B, H * W, C)
        x = shortcut + self.drop_path(x)

        # Feed-forward network (FFN)
        x = x + self.drop_path(self.mlp(self.norm2(x)))

        return x
```

#### Swin Transformer Block's Forward Pass
There are 4 key steps:

1. **Cyclic shift**:
The feature map is partitioned into windows via `window_partition`. A **cyclic shift** is then applied to the partitions. Cyclic shift is done by moving elements (in this case, partitions) in a sequence to the left or right, and wrapping around the elements that go off one end back to the other end. This process changes the positions of the elements relative to each other but keeps the sequence otherwise intact. For example, if you cyclically shift the sequence `A, B, C, D` to the right by one position, it becomes `D, A, B, C`.

Cyclic shift allows the model to capture relationships between adjacent windows, enhancing its ability to learn spatial contexts beyond the local scope of individual windows.

2. **Windowed attention**: Perform attention using window-based multi-head self attention (W-MSA) module.

3. **Merge Patches**: Patches are merged via `PatchMerging`.

4. **Reverse cyclic shift**: After attention is done, the window partitioning is undone via `reverse_window`, and the cyclic shift operation is reversed, so that the feature map retains its original form.

```python
class WindowAttention(nn.Module):
    """
    Args:
        dim (int): Number of input channels.
        window_size (tuple[int]): The height and width of the window.
        num_heads (int): Number of attention heads.
        qkv_bias (bool, optional):  If True, add a learnable bias to query, key, value. Default: True
        qk_scale (float | None, optional): Override default qk scale of head_dim ** -0.5 if set
        attn_drop (float, optional): Dropout ratio of attention weight. Default: 0.0
        proj_drop (float, optional): Dropout ratio of output. Default: 0.0
    """

    def __init__(
        self,
        dim,
        window_size,
        num_heads,
        qkv_bias=True,
        qk_scale=None,
        attn_drop=0.0,
        proj_drop=0.0,
    ):

        super().__init__()
        self.dim = dim
        self.window_size = window_size  # Wh, Ww
        self.num_heads = num_heads
        head_dim = dim // num_heads
        self.scale = qk_scale or head_dim**-0.5

        # define a parameter table of relative position bias
        self.relative_position_bias_table = nn.Parameter(
            torch.zeros((2 * window_size[0] - 1) * (2 * window_size[1] - 1), num_heads)
        )  # 2*Wh-1 * 2*Ww-1, nH

        # get pair-wise relative position index for each token inside the window
        coords_h = torch.arange(self.window_size[0])
        coords_w = torch.arange(self.window_size[1])
        coords = torch.stack(torch.meshgrid([coords_h, coords_w]))  # 2, Wh, Ww
        coords_flatten = torch.flatten(coords, 1)  # 2, Wh*Ww
        relative_coords = (
            coords_flatten[:, :, None] - coords_flatten[:, None, :]
        )  # 2, Wh*Ww, Wh*Ww
        relative_coords = relative_coords.permute(1, 2, 0).contiguous()  # Wh*Ww, Wh*Ww, 2
        relative_coords[:, :, 0] += self.window_size[0] - 1  # shift to start from 0
        relative_coords[:, :, 1] += self.window_size[1] - 1
        relative_coords[:, :, 0] *= 2 * self.window_size[1] - 1
        relative_position_index = relative_coords.sum(-1)  # Wh*Ww, Wh*Ww
        self.register_buffer("relative_position_index", relative_position_index)

        self.qkv = nn.Linear(dim, dim * 3, bias=qkv_bias)
        self.attn_drop = nn.Dropout(attn_drop)
        self.proj = nn.Linear(dim, dim)
        self.proj_drop = nn.Dropout(proj_drop)

        trunc_normal_(self.relative_position_bias_table, std=0.02)
        self.softmax = nn.Softmax(dim=-1)

    def forward(self, x, mask=None):
        """
        Args:
            x: input features with shape of (num_windows*B, N, C)
            mask: (0/-inf) mask with shape of (num_windows, Wh*Ww, Wh*Ww) or None
        """
        B_, N, C = x.shape
        qkv = (
            self.qkv(x)
            .reshape(B_, N, 3, self.num_heads, C // self.num_heads)
            .permute(2, 0, 3, 1, 4)
        )
        q, k, v = (
            qkv[0],
            qkv[1],
            qkv[2],
        )  # make torchscript happy (cannot use tensor as tuple)

        q = q * self.scale
        attn = q @ k.transpose(-2, -1)

        relative_position_bias = self.relative_position_bias_table[
            self.relative_position_index.view(-1)
        ].view(
            self.window_size[0] * self.window_size[1],
            self.window_size[0] * self.window_size[1],
            -1,
        )  # Wh*Ww,Wh*Ww,nH
        relative_position_bias = relative_position_bias.permute(
            2, 0, 1
        ).contiguous()  # nH, Wh*Ww, Wh*Ww
        attn = attn + relative_position_bias.unsqueeze(0)

        if mask is not None:
            nW = mask.shape[0]
            attn = attn.view(B_ // nW, nW, self.num_heads, N, N) + mask.unsqueeze(
                1
            ).unsqueeze(0)
            attn = attn.view(-1, self.num_heads, N, N)
            attn = self.softmax(attn)
        else:
            attn = self.softmax(attn)

        attn = self.attn_drop(attn)

        x = (attn @ v).transpose(1, 2).reshape(B_, N, C)
        x = self.proj(x)
        x = self.proj_drop(x)
        return x
```


### Window Attention
`WindowAttention` is a window-based multi-head self attention (W-MSA) module with relative position bias. This can be used for both shifted and non-shifted windows.

```python
class PatchMerging(nn.Module):
    r"""Patch Merging Layer.

    Args:
        input_resolution (tuple[int]): Resolution of input feature.
        dim (int): Number of input channels.
        norm_layer (nn.Module, optional): Normalization layer.  Default: nn.LayerNorm
    """

    def __init__(self, input_resolution, dim, norm_layer=nn.LayerNorm):
        super().__init__()
        self.input_resolution = input_resolution
        self.dim = dim
        self.reduction = nn.Linear(4 * dim, 2 * dim, bias=False)
        self.norm = norm_layer(4 * dim)

    def forward(self, x):
        """
        x: B, H*W, C
        """
        H, W = self.input_resolution
        B, L, C = x.shape
        assert L == H * W, "input feature has wrong size"
        assert H % 2 == 0 and W % 2 == 0, f"x size ({H}*{W}) are not even."

        x = x.view(B, H, W, C)

        x0 = x[:, 0::2, 0::2, :]  # B H/2 W/2 C
        x1 = x[:, 1::2, 0::2, :]  # B H/2 W/2 C
        x2 = x[:, 0::2, 1::2, :]  # B H/2 W/2 C
        x3 = x[:, 1::2, 1::2, :]  # B H/2 W/2 C
        x = torch.cat([x0, x1, x2, x3], -1)  # B H/2 W/2 4*C
        x = x.view(B, -1, 4 * C)  # B H/2*W/2 4*C

        x = self.norm(x)
        x = self.reduction(x)

        return x
```

### Patch Merging Layer
Patch merging method is used for downsampling. It is used to reduce the spatial dimensions of the feature map, similar to pooling in traditional convolutional neural networks (CNNs). It helps in building hierarchical feature representations by progressively increasing the receptive field and reducing the spatial resolution.

```python
from datasets import load_dataset
from transformers import AutoImageProcessor, SwinForImageClassification
import torch

model = SwinForImageClassification.from_pretrained(
    "microsoft/swin-tiny-patch4-window7-224"
)
image_processor = AutoImageProcessor.from_pretrained(
    "microsoft/swin-tiny-patch4-window7-224"
)

dataset = load_dataset("huggingface/cats-image")
image = dataset["test"]["image"][0]

inputs = image_processor(image, return_tensors="pt")

with torch.no_grad():
    logits = model(**inputs).logits

predicted_label_id = logits.argmax(-1).item()
predicted_label_text = model.config.id2label[predicted_label_id]

print(predicted_label_text)
```


## Try it out
You can find the 🤗 documentation for Swin [here](https://huggingface.co/docs/transformers/model_doc/swin).

### Usage of pretrained Swin model for classification
Here is how to use Swin model to classify a cat image into one of the 1,000 ImageNet classes:

```py
from datasets import load_dataset
from transformers import AutoImageProcessor, SwinForImageClassification
import torch

model = SwinForImageClassification.from_pretrained(
    "microsoft/swin-tiny-patch4-window7-224"
)
image_processor = AutoImageProcessor.from_pretrained(
    "microsoft/swin-tiny-patch4-window7-224"
)

dataset = load_dataset("huggingface/cats-image")
image = dataset["test"]["image"][0]

inputs = image_processor(image, return_tensors="pt")

with torch.no_grad():
    logits = model(**inputs).logits

predicted_label_id = logits.argmax(-1).item()
predicted_label_text = model.config.id2label[predicted_label_id]

print(predicted_label_text)
```

---

## Model Comparision Tables

**Table 1: Swin Variants Comparison**  
| Model          | Params | FLOPs (G) | ImageNet Top-1 | Key Features |
|----------------|---------|-----------|------------------|--------------|
| **Swin-T**     | 28M     | 4.5       | 81.3%            | Base hierarchical window design |
| **Swin-S**     | 50M     | 8.7       | 83.0%            | Mid-scale variant |
| **Swin-B**     | 88M     | 15.4      | 83.5%            | Outperforms DeiT-B by +1.5% |
| **SwinV2-G**   | 3B      | -         | 84.0% (ImageNet-V2) | Supports 1,536×1,536 resolution |
| **CSWin-T**    | 23M     | 4.3       | 82.7%            | Cross-shaped window attention |


**Table 2: CNN vs Transformer Comparison**  
| Model             | Params | FLOPs (G) | ImageNet Top-1 | Key Traits |
|-------------------|---------|-----------|------------------|------------|
| **ResNet-50**     | 25M     | 4.1       | 76.1%            | CNN baseline |
| **EfficientNet-B4** | 19M   | 4.2       | 82.9%            | Mobile-optimized |
| **ConvNeXt-T**    | 28M     | 4.5       | 82.1%            | Modern CNN SOTA |
| **DeiT-S**        | 22M     | 4.6       | 79.8%            | ViT baseline |
| **Swin-T**        | 28M     | 4.5       | 81.3%            | Hierarchical windows |
| **CSWin-T**       | 23M     | 4.3       | 82.7%            | Cross-shaped windows |


print(predicted_label_text)
```


