# AR-foundation

Autor: Eric Bermúdez Hernández

Email: alu0101517476@ull.edu.es

---

## 1. Introducción y Objetivos
El objetivo de este proyecto ha sido desarrollar una aplicación de Realidad Aumentada (RA) para dispositivos Android utilizando **Unity 6** y el framework **AR Foundation** junto con **Google ARCore**.

La funcionalidad principal de la aplicación consiste en:
- Detección de planos horizontales en el mundo real.
  
- Instanciación de objetos virtuales ("paquetes") al tocar dichas superficies.
  
- Interacción con los objetos: al tocar un paquete virtual, este se recoge (desaparece).
  
- Interfaz de Usuario (UI): Visualización en tiempo real de un contador de paquetes recogidos.

## 2. Configuración del Entorno (Project Setup)

### 2.1. Cambio de Plataforma
Para asegurar la compatibilidad con dispositivos móviles, se configuró el entorno de compilación para Android:
* **File > Build Settings:** Se seleccionó la plataforma **Android** y se ejecutó "Switch Platform".

### 2.2. Instalación de Paquetes (Package Manager)
Se instalaron las librerías necesarias desde el *Unity Registry*:
* **AR Foundation:** Framework base para el desarrollo multiplataforma de RA.
* **Google ARCore XR Plugin:** Plugin específico para soportar la tecnología ARCore en Android.
* **TextMeshPro:** Importación de *TMP Essentials* para la gestión de la interfaz de texto.

### 2.3. Activación de XR
* En **Project Settings > XR Plug-in Management**, se activó el proveedor **Google ARCore** en la pestaña de Android.

---

## 3. Construcción de la Escena

Se eliminó la cámara por defecto y se estructuró la jerarquía de la siguiente manera:

### 3.1. Componentes AR (XR Origin)
En Unity 6, se utilizó el objeto **XR Origin (Mobile AR)** en lugar del antiguo *AR Session Origin*.
* **XR Origin:** Objeto padre que maneja la cámara y las coordenadas del mundo real.
* **AR Session:** Componente encargado de gestionar el ciclo de vida de la experiencia de RA.

Al objeto **XR Origin** se le añadieron los siguientes componentes esenciales:
* **AR Plane Manager:** Para la detección y visualización de superficies planas. Se asignó un *AR Default Plane* como prefab de visualización.
* **AR Raycast Manager:** Para permitir lanzar rayos invisibles que detecten la intersección con los planos detectados.

### 3.2. Creación del "Paquete" (Prefab)
Se creó un objeto interactivo para representar los paquetes:
* **Geometría:** Cubo escalado a `0.2, 0.2, 0.2` (20 cm).
* **Física:** Componente `Rigidbody` con la propiedad `Is Kinematic` activada (para evitar que caiga al vacío).
* **Etiquetado:** Se creó y asignó el Tag **"Paquete"** para identificarlo mediante código.
* El objeto se convirtió en un **Prefab** para poder instanciarlo múltiples veces.

### 3.3. Interfaz de Usuario (UI)
* **Canvas:** Configurado con *UI Scale Mode: Scale With Screen Size*.
* **Texto:** Se utilizó un componente *TextMeshProUGUI* para mostrar el contador: "Paquetes: 0".

---

## 4. Lógica de Programación (Scripting)

Se implementó el script `GameController.cs` para gestionar la lógica. El script maneja dos tipos de interacción mediante toques en pantalla:

1.  **Raycast Físico:** Detecta si el usuario ha tocado un objeto virtual existente (el paquete). Si es así, lo destruye y suma puntos.
2.  **Raycast AR:** Si no se toca un paquete, busca un plano AR en el mundo real para instanciar un nuevo paquete.

### Código final implementado:
```C#
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;
using TMPro;

public class GameController : MonoBehaviour
{
    [Header("Configuración AR")]
    public ARRaycastManager raycastManager;
    public GameObject paquetePrefab;

    [Header("Interfaz")]
    public TextMeshProUGUI textoContador;

    private int paquetesRecogidos = 0;
    private List<ARRaycastHit> hits = new List<ARRaycastHit>();

    void Update()
    {
        if (Input.touchCount == 0) return;

        Touch touch = Input.GetTouch(0);

        if (touch.phase == TouchPhase.Began)
        {
            HandleTouch(touch.position);
        }
    }

    void HandleTouch(Vector2 touchPosition)
    {
        // Uso explícito de UnityEngine.Camera y Physics para evitar ambigüedades
        Ray ray = UnityEngine.Camera.main.ScreenPointToRay(touchPosition);
        UnityEngine.RaycastHit hitFisico;

        // 1. Intentar recoger paquete (Física estándar)
        if (UnityEngine.Physics.Raycast(ray, out hitFisico))
        {
            if (hitFisico.transform.CompareTag("Paquete"))
            {
                RecogerPaquete(hitFisico.transform.gameObject);
                return;
            }
        }

        // 2. Intentar poner paquete (Raycast AR)
        if (raycastManager.Raycast(touchPosition, hits, TrackableType.PlaneWithinPolygon))
        {
            Pose hitPose = hits[0].pose;
            Instantiate(paquetePrefab, hit

```

El siguiente vídeo muestra como funciona la aplicación en un dispositivo Android:

![Vídeo](Edit.gif)
