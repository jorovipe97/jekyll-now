---
layout: post
title: Usando Replacement Shaders
published: false
---

En esta ocación vamos a hablar un poco acerca del uso de los ReplacementShaders en unity, ¿Que son? ¿como se podrian usar?

# ¿Como funciona?
Segun la documentación de unity:

> It works like this: the camera renders the scene as it normally would. the objects still use their materials, but the actual shader that ends up being used is changed.

En otras palabras, esta caracteristica consiste en escribir un **Shader** que remplazará el **Shader** usado por los objetos de la escena.

![](https://imgur.com/gbJXkkf.gif)

El componente encargado de reemplazar los **Shaders** de todos los objetos de la escena por el **Shader** que escribimos es la **Camera**.

Analicemos el siguiente ejemplo bajo los ojos de la documentación oficial

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[ExecuteInEditMode]
public class ReplacementShaderEffect : MonoBehaviour {

    public Shader ReplacementShader;
    public string Tag = "RenderType";

    public Color OverDrawColor;
    
    // Called when the script is loaded or a value is changed in the inspector (Called in the editor only).
    private void OnValidate()
    {
        Shader.SetGlobalColor("_OverDrawColor", OverDrawColor);
    }

    private void OnEnable()
    {
        if (ReplacementShader != null)
        {
            GetComponent<Camera>().SetReplacementShader(ReplacementShader, Tag);
        }
    }

    private void OnDisable()
    {
        GetComponent<Camera>().ResetReplacementShader();
    }
}
```

# Bibliografia
- [Rendering with replacement shaders](https://docs.unity3d.com/Manual/SL-ShaderReplacement.html)
- [Using replacement shaders](https://www.youtube.com/watch?v=Tjl8jP5Nuvc)
- [Blending by learnopengl ](https://learnopengl.com/#!Advanced-OpenGL/Blending)
- [Blending by Jaanus Jaggo](https://cglearn.codelight.eu/pub/computer-graphics/blending)
