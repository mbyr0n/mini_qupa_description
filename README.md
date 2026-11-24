# Descripción del Qupa Mini para ROS 2

Edita `urdf/qupa.urdf.xacro` y vuelve a expandirlo para actualizar `urdf/qupa.urdf`. Las cinco mallas visuales `.dae` usan metros y conservan los colores del OBJ/MTL. El STEP se usó para identificar las piezas móviles y medir sus ejes. La geometría de la cámara, el espejo y los sensores forma parte de la malla visual del cuerpo.

La versión 0.3.0 se basa en los originales `cad/Ensable MiniQupa.step`, `.obj` y `.mtl`, recibidos el 24 de noviembre de 2026. Incorpora los materiales visuales actualizados y los marcos de montaje de los sensores, motores, cámara, espejo, placa superior y difusor en `urdf/component_frames.xacro`.

Los archivos STEP y OBJ se guardan mediante Git LFS. Para descargar sus contenidos completos necesitas tener Git LFS instalado. El MTL conserva el nombre que referencia el OBJ. Los originales CAD sirven como fuente de diseño, y el paquete ROS instala las mallas DAE y los archivos URDF/Xacro.

## Ver el robot en RViz

En una terminal con ROS 2 ya cargado, clona el paquete en tu workspace:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git lfs install
git clone https://github.com/mbyr0n/mini_qupa_description.git
git -C mini_qupa_description lfs pull
```

También puedes descomprimir el ZIP y colocar `mini_qupa_description` en `~/ros2_ws/src`. Después ejecuta desde tu workspace:

```bash
cd ~/ros2_ws
rosdep install --from-paths src/mini_qupa_description --ignore-src -y
colcon build --packages-select mini_qupa_description --symlink-install
source install/setup.bash
ros2 launch urdf_launch display.launch.py \
  urdf_package:=mini_qupa_description \
  urdf_package_path:=urdf/qupa.urdf.xacro
```

Usa los controles `left_wheel_joint` y `right_wheel_joint` para girar cada rueda. En RViz puedes elegir `base_footprint` como *Fixed Frame* y acercar la cámara al robot.

Para regenerar el URDF después de editar el Xacro, ejecuta desde la carpeta `mini_qupa_description`:

```bash
xacro urdf/qupa.urdf.xacro -o urdf/qupa.urdf
```

Para generar el URDF con rutas absolutas a las mallas, puedes añadir `mesh_dir:="file://$(pwd)/meshes"` al comando anterior.

## Medidas y marcos

| Medida | Valor en el STEP |
| --- | ---: |
| Diámetro de rueda motriz | 32 mm |
| Separación entre centros de las bandas de rodadura | 79,5 mm |
| Ancho de la banda de rodadura | 6 mm |
| Ancho máximo de cada conjunto de rueda | 8,5 mm |
| Diámetro de cada bola de apoyo | 12,7 mm |
| Ancho máximo del ensamble | 108,4 mm |
| Altura del ensamble sobre el plano nominal de rodadura | 167,3 mm |

`base_link` está a mitad de camino entre las ruedas y a 16 mm sobre `base_footprint`. El modelo usa +X hacia delante, +Y a la izquierda y +Z hacia arriba. Se tomó como frente el lado del sensor de color inferior, que corresponde a +Z en el CAD. Los centros motrices conservan sus desplazamientos longitudinales de −0,05 mm a la izquierda y +0,05 mm a la derecha. La bola delantera queda 0,5 mm por encima del plano nominal en el CAD.

El modelo describe la geometría, las articulaciones y colisiones aproximadas para visualización. Las bolas se representan como apoyos fijos. Para simular el movimiento en Gazebo quedan pendientes las masas e inercias, el contacto de los apoyos y el controlador. El correo del mantenedor en `package.xml` es un marcador de posición.

## Ubicaciones revisadas en el CAD

Todas las posiciones de esta tabla están expresadas en milímetros respecto a `base_link`. Suma 16 mm a Z para obtener la altura sobre el plano de rodadura. El Xacro usa metros. Las pequeñas diferencias entre lados y alturas se conservaron según el STEP.

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

Los marcos de los motores están en los orígenes de montaje del STEP, sobre sus ejes de salida. Sus ejes Y son paralelos a los de las ruedas. Los cuerpos permanecen fijos a `base_link` y las ruedas usan las articulaciones continuas existentes.

El marco del TCS34725 se coloca en el centro de la cara del encapsulado que mira al suelo. Su eje +X apunta hacia abajo. El centro del área fotosensible se aproxima mediante el centro del encapsulado del CAD.

Cada marco Sharp GP2Y0E03 se coloca en el punto medio de las dos ventanas de lente simplificadas. Su +X apunta hacia fuera, con +Z hacia arriba. Los ángulos alrededor de Z son +15° y -15° para los frontales, +50° y -50° para los diagonales delanteros, +90° y -90° para los laterales, y +150° y -150° para los traseros. Estas posiciones son referencias nominales de montaje. El origen efectivo de medición debe ajustarse con la calibración del sensor.

`camera_link` se coloca en el centro de la apertura frontal de la lente y apunta con +X hacia arriba, al espejo. `camera_optical_frame` aplica la convención óptica ROS, con +Z hacia el espejo. La apertura del CAD aproxima el centro óptico, y la rotación de la imagen debe contrastarse con la configuración del controlador y la calibración. Los argumentos Xacro `camera_optical_xyz` y `camera_optical_rpy` permiten ajustar su posición y orientación relativas a `camera_link`.

`mirror_link` usa el origen de montaje del espejo sobre su eje de simetría. `top_plate_link` marca el centro de la cara plana superior donde se puede montar el AprilTag. `diffuser_link` usa el origen de montaje del conjunto superior. Sus ejes son paralelos a `base_link`. El marcador AprilTag necesita su propia posición y orientación de impresión cuando se defina el montaje.

La IMU continúa pendiente de identificar por modelo y ubicación del módulo. Los dos esquemáticos revisados, `SCH_powerpcb_FINAL.json` y `SCH_QupaNewDesignWOUTMPU_FINAL.json`, no incluyen una IMU identificable, y la hoja `OWMPU copy` contiene únicamente el marco del dibujo.

Convenciones de referencia en [REP 103](https://www.ros.org/reps/rep-0103.html), [sensor_msgs/Range](https://github.com/ros2/common_interfaces/blob/rolling/sensor_msgs/msg/Range.msg) y [sensor_msgs/Image](https://github.com/ros2/common_interfaces/blob/rolling/sensor_msgs/msg/Image.msg). Información sobre [Git LFS](https://docs.github.com/en/repositories/working-with-files/managing-large-files/configuring-git-large-file-storage).
