# Modelo del Mini Qupa

[Volver al README](../README.md)

La versión 0.3.0 se basa en los originales `Ensable MiniQupa.step`, `.obj` y `.mtl` incorporados el 24 de noviembre de 2026. Están disponibles en [`cad/`](../cad/). Los marcos de montaje se definen en [`urdf/component_frames.xacro`](../urdf/component_frames.xacro).

## Dimensiones

| Medida | Valor en el STEP |
| --- | ---: |
| Diámetro de rueda motriz | 32 mm |
| Separación entre centros de las bandas de rodadura | 79,5 mm |
| Ancho de la banda de rodadura | 6 mm |
| Ancho máximo de cada conjunto de rueda | 8,5 mm |
| Diámetro de cada bola de apoyo | 12,7 mm |
| Ancho máximo del ensamble | 108,4 mm |
| Altura del ensamble sobre el plano nominal de rodadura | 167,3 mm |

## Convención de coordenadas

El modelo usa metros en el URDF/Xacro y ejes +X hacia delante, +Y hacia la izquierda y +Z hacia arriba. El frente corresponde al lado del sensor de color inferior, que es +Z en el CAD.

`base_link` está a mitad de camino entre las ruedas y a 16 mm sobre `base_footprint`. La conversión desde las coordenadas CAD expresadas en milímetros es la siguiente.

```text
x_ROS = z_CAD / 1000
y_ROS = x_CAD / 1000
z_ROS = y_CAD / 1000 - 0.010
```

Los centros motrices conservan sus desplazamientos longitudinales de -0,05 mm a la izquierda y +0,05 mm a la derecha. La bola delantera queda 0,5 mm por encima del plano nominal de rodadura en el CAD. Las bolas se representan mediante uniones fijas.

## Ubicaciones de los componentes

Todas las posiciones de la tabla están expresadas en **milímetros respecto a `base_link`**. Suma 16 mm a Z para obtener la altura sobre el plano de rodadura. Las pequeñas diferencias entre lados y alturas se conservaron según el STEP.

| Marco | X (mm) | Y (mm) | Z (mm) |
| --- | ---: | ---: | ---: |
| `left_motor_link` | -0.050 | 24.200 | 0.000 |
| `right_motor_link` | 0.050 | -24.200 | 0.000 |
| `floor_sensor_link` | 10.925 | -0.397 | -10.406 |
| `ir_front_left_link` | 41.076 | 11.537 | 55.250 |
| `ir_front_right_link` | 41.153 | -11.896 | 55.533 |
| `ir_left_front_link` | 27.237 | 33.125 | 55.250 |
| `ir_right_front_link` | 27.055 | -32.960 | 54.850 |
| `ir_left_link` | -0.821 | 42.621 | 55.250 |
| `ir_right_link` | -0.573 | -42.504 | 55.350 |
| `ir_rear_left_link` | -37.457 | 21.080 | 54.450 |
| `ir_rear_right_link` | -37.328 | -20.564 | 55.350 |
| `camera_link` | -0.202 | -0.151 | 109.231 |
| `mirror_link` | 0.000 | 0.000 | 126.000 |
| `top_plate_link` | 0.000 | 0.000 | 151.343 |
| `diffuser_link` | 0.000 | 0.000 | 90.850 |

### Motores

Los marcos de los motores están en los orígenes de montaje del STEP, sobre sus ejes de salida. Sus ejes Y son paralelos a los de las ruedas. Los cuerpos permanecen fijos a `base_link` y las ruedas usan las articulaciones continuas `left_wheel_joint` y `right_wheel_joint`.

### Sensor de piso

`floor_sensor_link` se coloca en el centro de la cara del encapsulado TCS34725 que mira al suelo. Su eje +X apunta hacia abajo. El centro del área fotosensible se aproxima mediante el centro del encapsulado del CAD.

### Sensores de distancia

Cada marco Sharp GP2Y0E03 se coloca en el punto medio de las dos ventanas de lente simplificadas. Su +X apunta hacia fuera, con +Z hacia arriba.

| Grupo | Giro alrededor de Z |
| --- | --- |
| Frontales | +15° a la izquierda y -15° a la derecha |
| Diagonales delanteros | +50° a la izquierda y -50° a la derecha |
| Laterales | +90° a la izquierda y -90° a la derecha |
| Traseros | +150° a la izquierda y -150° a la derecha |

Estas posiciones son referencias nominales de montaje. El origen efectivo de medición debe ajustarse con la calibración del sensor.

### Cámara y espejo

`camera_link` se coloca en el centro de la apertura frontal de la lente OV5647 y apunta con +X hacia arriba, al espejo. `camera_optical_frame` aplica la convención óptica ROS, con +Z hacia el espejo. La apertura del CAD aproxima el centro óptico, y la rotación de la imagen debe contrastarse con la configuración del controlador y la calibración.

Los argumentos Xacro siguientes permiten ajustar la posición y orientación relativas a `camera_link`.

| Argumento | Valor predeterminado | Unidad |
| --- | --- | --- |
| `camera_optical_xyz` | `0 0 0` | m |
| `camera_optical_rpy` | `-1.5707963267948966 0 -1.5707963267948966` | rad |

`mirror_link` usa el origen de montaje del espejo sobre su eje de simetría. Sus ejes son paralelos a `base_link`.

### Placa superior y difusor

`top_plate_link` marca el centro de la cara plana superior donde se puede montar el AprilTag. El marcador necesita su propia posición y orientación de impresión cuando se defina el montaje.

`diffuser_link` usa el origen de montaje del conjunto superior. Los ejes de ambos marcos son paralelos a `base_link`.

### IMU

La IMU continúa pendiente de identificar por modelo y ubicación del módulo. Los esquemáticos revisados, `SCH_powerpcb_FINAL.json` y `SCH_QupaNewDesignWOUTMPU_FINAL.json`, contienen los circuitos de alimentación e interfaz. La hoja `OWMPU copy` contiene únicamente el marco del dibujo. El marco `imu_link` se definirá al identificar el módulo y su montaje.

## Geometría y colisiones

Las cinco mallas visuales DAE conservan los colores del OBJ/MTL y utilizan metros. El cuerpo, la electrónica, la cámara, el espejo y los sensores fijos se representan en `base_link.dae`. Cada rueda y cada bola tienen su propia malla.

Las colisiones del cuerpo usan cajas y cilindros. Las ruedas usan cilindros y los apoyos usan esferas. Estas formas aproximan el ensamble para visualizar sus envolventes.

Para preparar una simulación dinámica quedan pendientes las masas e inercias, el contacto de los apoyos y el controlador.

## Generación del URDF

Desde la carpeta del paquete, genera el URDF después de editar cualquiera de los archivos Xacro.

```bash
xacro urdf/qupa.urdf.xacro -o urdf/qupa.urdf
```

Para generar rutas absolutas a las mallas, ejecuta desde esa misma carpeta.

```bash
xacro urdf/qupa.urdf.xacro \
  mesh_dir:="file://$(pwd)/meshes" \
  -o urdf/qupa.urdf
```

## Referencias

- [Convenciones de unidades y ejes, REP 103](https://www.ros.org/reps/rep-0103.html)
- [Mensaje sensor_msgs/Range](https://github.com/ros2/common_interfaces/blob/rolling/sensor_msgs/msg/Range.msg)
- [Mensaje sensor_msgs/Image](https://github.com/ros2/common_interfaces/blob/rolling/sensor_msgs/msg/Image.msg)
