---
layout: post
title: Usando Replacement Shaders
published: false
---

En esta ocación vamos a hablar un poco acerca del uso de los ReplacementShaders en unity, ¿Que son? ¿como se podrian usar?

Usando esta tecnica podemos lograr efectos como este:

<iframe width="560" height="315" src="https://www.youtube.com/embed/kfISz5QA20k" frameborder="0" gesture="media" allowfullscreen></iframe>

# ¿Como funciona?
Segun la documentación de unity:

> It works like this: the camera renders the scene as it normally would. the objects still use their materials, but the actual shader that ends up being used is changed.

En otras palabras, esta caracteristica consiste en escribir un **Shader** que remplazará el **Shader** usado por los objetos de la escena.

![](https://imgur.com/gbJXkkf.gif)

El componente encargado de reemplazar los **Shaders** de todos los objetos de la escena por el **Shader** que escribimos es la **Camera**.

# Usando el reemplazo de shaders.
Analicemos como usar el remplazo de Shaders con algo de codigo:

Para usar esta tecnica nececitamos dos cosas:
1. Escribir un Script MonoBehaviour que le comunique a la **Camera** cual es el shader de reemplazamiento y cuando lo debe usar.
2. Escribir Shader que reemplazara el Shader de los objetos de la escena (Shader de reemplazamiento)

## Script MonoBehaviour
Este es solo un Script de ejemplo, las dos partes importante a entender de el son los llamados a los metodos: **SetReplacementShader** y **ResetReplacementShader**

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
        Shader.SetGlobalColor("_OverDrawColor", OverDrawColor); // This line Sets a shader property without a Material.
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

### SetReplacementShader(Shader replacementShader, String tag)
```c#
GetComponent<Camera>().SetReplacementShader(ReplacementShader, "");
```
Cuando este metodo es llamado, se reemplazara el **Shader** que se usara para cada objeto de la escena basado en el segundo argumento, el cual se explicara en detalle mas adelante.

### Camera.ResetReplacementShader();
```c#
GetComponent<Camera>().SetReplacementShader(ReplacementShader, "");
```
Cuando este metodo se llama, elimina los **Replacement shaders** y deja los **Shaders** que tenia cada objeto originalmente. 

## Shader de reemplazamiento
Podemos ver que el **Shader de reemplazamiento** tiene 2 subshaders con **tags** diferentes, entonces ¿Cual de los dos subshaders sera el que se usara para hacer el reemplazo?

Esto tiene dos respuestas:

Se usará el primer **Subshader** si el segundo argumento de:
```c#
Camera.SetReplacementShader(ReplacementShader, "");
```
es un string vacio.

Por otro lado se usará el **Subshader** que tenga el mismo valor que el objeto en cuestion al que se le reemplazara el **Shader** en el **tag** especificado por el segundo argumento:
```c#
Camera.SetReplacementShader(ReplacementShader, "RenderType");
```
Es decir, supongamos que tenemos una esfera a la cual se le reemplazará el **Shader**, si dicho shader tiene el **tag**: "RenderType"="Opaque" entonces se usara el **Subshader** del **Shader de reemplazamiento** que tenga ese mismo tag con ese mismo valor

Si el objeto al que se le reemplazara el shader no tiene especificado en su **Shader original** dicho **tag** o si tiene un valor que no ha sido escrito en el **Replacement Shader** dicho objeto no se renderizara.

Por ejemplo si tenemos una esfera con el tag "RenderType"="AnCrazyValue" entonces dicha esfera no sera renderizada cuando llamemos al metodo SetReplacementShader.

NOTA: El material original del objeto no es eliminado cuando se reemplaza el **shader**, es decir que podemos acceder a las propiedades de dichos materiales desde el **Subshader** del **ReplacementShader**, para hacer esto debemos escribir en el **ReplacementShader** una propiedad con el mismo nombre de la propiedad del matarial del objeto al cual estamos remplazando el **Shader**

```ShaderLab
Shader "Hidden/A Beautiful Shader"
{
	Properties
	{
		_Color("Color", Color) = (1,1,1,1)
	}

	SubShader
	{
		Tags
		{
			"RenderType"="Opaque"
		}
        
		Pass
		{
			CGPROGRAM
			// CG Code goes here
			ENDCG
		}
	}

	SubShader
	{
		Tags
		{
			"RenderType" = "Transparent"
		}
        
		Pass
		{
			CGPROGRAM
			// CG Code goes here
			ENDCG
		}
	}
}
```

# Bibliografia
- [Rendering with replacement shaders](https://docs.unity3d.com/Manual/SL-ShaderReplacement.html)
- [Using replacement shaders](https://www.youtube.com/watch?v=Tjl8jP5Nuvc)
- [Blending by learnopengl ](https://learnopengl.com/#!Advanced-OpenGL/Blending)
- [Blending by Jaanus Jaggo](https://cglearn.codelight.eu/pub/computer-graphics/blending)
