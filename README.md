# Descripción del Qupa Mini para ROS 2

Edita `urdf/qupa.urdf.xacro` y vuelve a expandirlo para actualizar `urdf/qupa.urdf`. Las cinco mallas visuales `.dae` usan metros y conservan los colores del OBJ/MTL. El STEP se usó para identificar las piezas móviles y medir sus ejes. La geometría de la cámara, el espejo y los sensores forma parte de la malla visual del cuerpo.

La versión 0.2.0 se basa en `Ensamble Mini Qupa.step`, `.obj` y `.mtl`, recibidos el 21 de noviembre de 2026. Incluye la tapa modificada, el soporte del espejo elevado 5 mm, las dos placas electrónicas y el XIAO ESP32-C6. La identificación de la IMU por modelo y referencia de placa queda pendiente para definir su marco `imu_link` y la orientación de sus ejes.

## Ver el robot en RViz

En una terminal con ROS 2 ya cargado, clona el paquete en tu workspace:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone https://github.com/mbyr0n/qupa_mini_description.git
```

También puedes descomprimir el ZIP y colocar `qupa_mini_description` en `~/ros2_ws/src`. Después ejecuta desde tu workspace:

```bash
cd ~/ros2_ws
rosdep install --from-paths src/qupa_mini_description --ignore-src -y
colcon build --packages-select qupa_mini_description --symlink-install
source install/setup.bash
ros2 launch urdf_launch display.launch.py \
  urdf_package:=qupa_mini_description \
  urdf_package_path:=urdf/qupa.urdf.xacro
```

Usa los controles `left_wheel_joint` y `right_wheel_joint` para girar cada rueda. En RViz puedes elegir `base_footprint` como *Fixed Frame* y acercar la cámara al robot.

Para regenerar el URDF después de editar el Xacro, ejecuta desde la carpeta `qupa_mini_description`:

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
