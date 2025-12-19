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
```
 
### 2. Implementación de Dijkstra
Se modificó el bloque principal (if __name__ == "__main__":) del script f1tenth_map.py para adaptar la planificación al entorno específico de BrandsHatch. Los cambios principales incluyen:

* **Selección del Algoritmo:** Se configuró el SearchFactory para inicializar el planificador con el algoritmo dijkstra.
* **Carga del Entorno:** Se actualizó la ruta del archivo de configuración a BrandsHatch_map.yaml para cargar la geometría real del circuito.
* **Definición de Coordenadas:** Se establecieron puntos de inicio (x_start, y_start) y meta (x_goal, y_goal) dentro de los límites de la pista para asegurar una convergencia exitosa del algoritmo.

    
```bash
if __name__ == "__main__":
    HERE = Path(__file__).resolve().parent
    yaml_path = HERE.parent / "Mapas-F1Tenth" / "BrandsHatch_map.yaml"
    downsample_factor = 4 # Ajusta este valor según lo que necesites

    x_start, y_start = -5.0, -10.0
    x_goal, y_goal = 40.0, -80.0

    map_bin, resolution, origin = load_map(yaml_path, downsample_factor)
    env = grid_from_map(map_bin)

    start = world_to_map(x_start, y_start, resolution, origin)
    goal = world_to_map(x_goal, y_goal, resolution, origin)

    print(f"Start (map): {start}, Goal (map): {goal}")
    planner = SearchFactory()("dijkstra", start=start, goal=goal, env=env)
    planner.run()

    cost, path, _ = planner.plan()
```


### 3. Generación de Waypoints (0.5m y 1.0m)
Dado que los algoritmos basados en rejilla (Grid-based) generan puntos dependientes del tamaño de la celda, se implementó una **función de interpolación lineal** (`ajustar_waypoints`) para garantizar la separación exacta exigida:
* **Prueba A:** Generación de un archivo `.csv` con waypoints separados exactamente **0.5 metros**.
* **Prueba B:** Generación de un archivo `.csv` con waypoints separados exactamente **1.0 metro**.


```bash
if path:
        # GENERAR Y GUARDAR TRAYECTORIA 0.5m
        path_05 = ajustar_waypoints(path, 0.5, resolution, origin)
        with open("dijkstra_05m.csv", "w") as f:
            f.write("x,y\n")
            for p in path_05: f.write(f"{p[0]},{p[1]}\n")
        print(f"Archivo 0.5m guardado ({len(path_05)} puntos)")

        # GENERAR Y GUARDAR TRAYECTORIA 1.0m
        path_10 = ajustar_waypoints(path, 1.0, resolution, origin)
        with open("dijkstra_10m.csv", "w") as f:
            f.write("x,y\n")
            for p in path_10: f.write(f"{p[0]},{p[1]}\n")
        print(f"Archivo 1.0m guardado ({len(path_10)} puntos)")

        # Mostrar animación final
        planner.plot.animation(path, "Dijkstra BrandsHatch", cost)
    else:
        print("No se encontró ruta.")
```
---

## 📊 Resultados Visuales

| Algoritmo | Distancia Waypoints | Visualización |
| :--- | :--- | :--- |
| **Dijkstra** | 0.5 m | ![Dijkstra 0.5m](https://github.com/EDCHC1234/F1Tenth-Trayectoria/blob/main/Figure_1.png) |
| **Dijkstra** | 1.0 m | ![Dijkstra 1.0m](enlace_a_tu_imagen_10) |

> **Nota:** El área gris representa el espacio de búsqueda expandido por el algoritmo antes de converger en la ruta óptima (línea verde).

## 🛠️ Cómo ejecutar
1. Asegurarse de tener el entorno virtual activo.
2. Ejecutar el script modificado:
   ```bash
   python3 f1tenth_map.py
