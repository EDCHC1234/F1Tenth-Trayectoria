# F1Tenth-Trayectoria
Planeación Global de Trayectorias 

# Tarea de Planificación de Movimiento - F1Tenth

Este repositorio contiene la implementación y comparación de algoritmos de planificación global aplicados al circuito de **BrandsHatch** y **RRT**.

## 🚀 Cambios Realizados

Para cumplir con los requerimientos de la tarea (Parte A), se modificó el script de ejecución principal `f1tenth_map.py` y se integraron funciones de procesamiento de trayectorias.

### 1. Configuración del Entorno (BrandsHatch)
* **Mapa:** Se configuró la carga del archivo `BrandsHatch_map.yaml`.
* **Resolución de Rejilla:** Se utilizó un `downsample_factor = 4.
* **Binarización:** Se ajustó el umbral de detección de obstáculos para asegurar que las líneas delgadas de la pista fueran detectadas correctamente por el algoritmo.

Antes de la linea: if __name__ == "__main__":, se aagrego el siguiente código

   ```bash
   def ajustar_waypoints(path, distancia_objetivo, resolution, origin):
    if not path: return []
    path_metros = []
    for x_map, y_map in reversed(path):
        x_w = x_map * resolution + origin[0]
        y_w = y_map * resolution + origin[1]
        path_metros.append(np.array([float(x_w), float(y_w)]))
    
    if len(path_metros) < 2: return path_metros
    nuevo_path = [path_metros[0]]
    p_actual = path_metros[0]
    
    for i in range(len(path_metros) - 1):
        p_siguiente = path_metros[i+1]
        while np.linalg.norm(p_siguiente - p_actual) >= distancia_objetivo:
            direccion = p_siguiente - p_actual
            direccion = direccion / np.linalg.norm(direccion)
            p_actual = p_actual + direccion * distancia_objetivo
            nuevo_path.append(p_actual.copy())
            
    return nuevo_path
´´´

### 2. Implementación de Dijkstra
Se utilizó la clase `Dijkstra` heredada de `AStar` para encontrar la ruta óptima.
* **Optimización:** Se desactivó la renderización frame a frame durante el cálculo para reducir el tiempo de ejecución en un 80%.

### 3. Generación de Waypoints (0.5m y 1.0m)
Dado que los algoritmos basados en rejilla (Grid-based) generan puntos dependientes del tamaño de la celda, se implementó una **función de interpolación lineal** (`ajustar_waypoints`) para garantizar la separación exacta exigida:
* **Prueba A:** Generación de un archivo `.csv` con waypoints separados exactamente **0.5 metros**.
* **Prueba B:** Generación de un archivo `.csv` con waypoints separados exactamente **1.0 metro**.

---

## 📊 Resultados Visuales

| Algoritmo | Distancia Waypoints | Visualización |
| :--- | :--- | :--- |
| **Dijkstra** | 0.5 m | ![Dijkstra 0.5m](/home/daniel08/planning.png) |
| **Dijkstra** | 1.0 m | ![Dijkstra 1.0m](enlace_a_tu_imagen_10) |

> **Nota:** El área gris representa el espacio de búsqueda expandido por el algoritmo antes de converger en la ruta óptima (línea verde).

## 🛠️ Cómo ejecutar
1. Asegurarse de tener el entorno virtual activo.
2. Ejecutar el script modificado:
   ```bash
   python3 f1tenth_map.py
