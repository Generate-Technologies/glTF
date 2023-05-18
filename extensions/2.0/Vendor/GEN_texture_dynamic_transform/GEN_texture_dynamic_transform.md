<!--
Copyright 2023 Generate Technologies.

Copyright 2017-2018 The Khronos Group Inc.
SPDX-License-Identifier: LicenseRef-KhronosSpecCopyright
-->

# GEN_texture_dynamic_transform

## Contributors

- Mark J. Grikis, Generate Technologies ([mark.jg@generate.design](mailto:mark.jg@generate.design))

Based on the `KHR_texture_transform` extension by Steven Vergenz, Microsoft

Copyright 2023 Generate Technologies.
Copyright 2017-2018 The Khronos Group Inc. All Rights Reserved. glTF is a trademark of The Khronos Group Inc.
See [Appendix](#appendix-full-khronos-copyright-statement) for full Khronos Copyright Statement.

## Status

Work in progress.

## Dependencies

Written against the glTF 2.0 spec.

## Overview

## Overview

The `GEN_texture_dynamic_transform` is an extension derived from `KHR_texture_transform`, and it enhances UV calculations by incorporating real-time object scaling. A new runtime shader variable, `objectScale`, is introduced to represent the runtime scale of the individual object instances. This `objectScale` multiplies the `scale` property during UV transformation, serving as a per-object property that reflects the current scale of the object. 

This extension was necessary as it provided a solution to maintain the integrity of textures during the 3D scaling of glTF meshes with repeating/tiled textures at runtime. Previously, scaling the meshes led to undesirable texture stretching. With the `objectScale` property, the textures now adjust dynamically with the scale transformations, preventing any distortions or stretching.

```hlsl
// Input UV coordinates
varying in vec2 uv;

// Defined properties
uniform vec2 offset, scale;
uniform vec3 objectScale;
uniform float rotation;

// Calculate translation matrix
mat3 translation = mat3(1.0, 0.0, 0.0, 0.0, 1.0, 0.0, offset.x, offset.y, 1.0);

// Calculate rotation matrix
mat3 rotate = mat3(
    cos(rotation), sin(rotation), 0.0,
   -sin(rotation), cos(rotation), 0.0,
                0.0,             0.0, 1.0
);

// Calculate scale matrix, incorporating object scale
mat3 scaleMatrix = mat3(scale.x * objectScale.x, 0.0, 0.0, 0.0, scale.y * objectScale.y, 0.0, 0.0, 0.0, 1.0);

// Combine transformations
mat3 matrix = translation * rotate * scaleMatrix;

// Apply transformation to UV coordinates
vec2 transformedUV = (matrix * vec3(uv, 1.0)).xy;

// Return transformed UV coordinates
return transformedUV;

```

This is equivalent to Unity's `Material#SetTextureOffset` and `Material#SetTextureScale`, or Three.js's `Texture#offset` and `Texture#repeat`. UV rotation is not widely supported as of today, but is included here for forward compatibility.

## glTF Schema Updates

The `GEN_texture_dynamic_transform` extension may be defined on `textureInfo` structures. It may contain the following properties:

| Name       | Type       | Default      | Description                                                                                                                               |
| ---------- | ---------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `offset`   | `array[2]` | `[0.0, 0.0]` | The offset of the UV coordinate origin as a factor of the texture dimensions.                                                             |
| `rotation` | `number`   | `0.0`        | Rotate the UVs by this many radians counter-clockwise around the origin. This is equivalent to a similar rotation of the image clockwise. |
| `scale`    | `array[2]` | `[1.0, 1.0]` | The scale factor applied to the components of the UV coordinates.                                                                         |
| `texCoord` | `integer`  |              | Overrides the textureInfo texCoord value if supplied, and if this extension is supported.                                                 |

Though this extension's values are unbounded, they will only produce sane results if the texture sampler's `wrap` mode is `REPEAT`, or if the result of the final UV transformation is within the range [0, 1] (i.e. negative scale settings and correspondingly positive offsets).

> **Implementation Note**: For maximum compatibility, it is recommended that exporters generate UV coordinate sets both with and without transforms applied, use the post-transform set in the texture `texCoord` field, then the pre-transform set with this extension. This way, if the extension is not supported by the consuming engine, the model still renders correctly. Including both will increase the size of the model, so if including the fallback UV set is too burdensome, either add this extension to `extensionsRequired` or use the same texCoord value in both places.

> **Implementation Note**: From the glTF core specification, the origin of the UV coordinates (0, 0) corresponds to the upper left corner of a texture image.

### Example JSON

This example utilizes only the lower left quadrant of the source image, rotated clockwise 90&deg;.

```json
{
  "materials": [
    {
      "emissiveTexture": {
        "index": 0,
        "extensions": {
          "GEN_texture_dynamic_transform": {
            "offset": [0, 1],
            "rotation": 1.57079632679,
            "scale": [0.5, 0.5]
          }
        }
      }
    }
  ]
}
```

This example inverts the T axis, effectively defining a bottom-left origin.

```json
{
  "materials": [
    {
      "emissiveTexture": {
        "index": 0,
        "extensions": {
          "GEN_texture_dynamic_transform": {
            "offset": [0, 1],
            "scale": [1, -1]
          }
        }
      }
    }
  ]
}
```

## Appendix: Full Khronos Copyright Statement

Copyright 2017-2018 The Khronos Group Inc.

Some parts of this Specification are purely informative and do not define requirements
necessary for compliance and so are outside the Scope of this Specification. These
parts of the Specification are marked as being non-normative, or identified as
**Implementation Notes**.

Where this Specification includes normative references to external documents, only the
specifically identified sections and functionality of those external documents are in
Scope. Requirements defined by external documents not created by Khronos may contain
contributions from non-members of Khronos not covered by the Khronos Intellectual
Property Rights Policy.

This specification is protected by copyright laws and contains material proprietary
to Khronos. Except as described by these terms, it or any components
may not be reproduced, republished, distributed, transmitted, displayed, broadcast
or otherwise exploited in any manner without the express prior written permission
of Khronos.

This specification has been created under the Khronos Intellectual Property Rights
Policy, which is Attachment A of the Khronos Group Membership Agreement available at
www.khronos.org/files/member_agreement.pdf. Khronos grants a conditional
copyright license to use and reproduce the unmodified specification for any purpose,
without fee or royalty, EXCEPT no licenses to any patent, trademark or other
intellectual property rights are granted under these terms. Parties desiring to
implement the specification and make use of Khronos trademarks in relation to that
implementation, and receive reciprocal patent license protection under the Khronos
IP Policy must become Adopters and confirm the implementation as conformant under
the process defined by Khronos for this specification;
see https://www.khronos.org/adopters.

Khronos makes no, and expressly disclaims any, representations or warranties,
express or implied, regarding this specification, including, without limitation:
merchantability, fitness for a particular purpose, non-infringement of any
intellectual property, correctness, accuracy, completeness, timeliness, and
reliability. Under no circumstances will Khronos, or any of its Promoters,
Contributors or Members, or their respective partners, officers, directors,
employees, agents or representatives be liable for any damages, whether direct,
indirect, special or consequential damages for lost revenues, lost profits, or
otherwise, arising from or in connection with these materials.

Vulkan is a registered trademark and Khronos, OpenXR, SPIR, SPIR-V, SYCL, WebGL,
WebCL, OpenVX, OpenVG, EGL, COLLADA, glTF, NNEF, OpenKODE, OpenKCAM, StreamInput,
OpenWF, OpenSL ES, OpenMAX, OpenMAX AL, OpenMAX IL, OpenMAX DL, OpenML and DevU are
trademarks of The Khronos Group Inc. ASTC is a trademark of ARM Holdings PLC,
OpenCL is a trademark of Apple Inc. and OpenGL and OpenML are registered trademarks
and the OpenGL ES and OpenGL SC logos are trademarks of Silicon Graphics
International used under license by Khronos. All other product names, trademarks,
and/or company names are used solely for identification and belong to their
respective owners.