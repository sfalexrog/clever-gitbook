# Работа с камерой

Для работы с основной камерой необходимо убедиться что она включена в файле `~/catkin_ws/src/clever/clever/launch/clever.launch`:

```xml
<arg name="main_camera" default="true"/>
```

Также нужно убедиться, что для камеры [указано корректное расположение и ориентация](camera_frame.md).

При изменении launch-файла необходимо перезапустить пакет `clever`:

```bash
sudo systemctl restart clever
```

Для мониторинга изображения с камеры можно использовать rqt или [web_video_server](web_video_server.md).

## Компьютерное зрение

Для реализации алгоритмов компьютерного зрения рекомендуется использовать предустановленную на [образ SD-карты](microsd_images.md) библиотеку [OpenCV](https://opencv.org).

### Python

Основная статья: http://wiki.ros.org/cv_bridge/Tutorials/ConvertingBetweenROSImagesAndOpenCVImagesPython.

Пример создания подписчика на топик с изображением с основной камеры для обрабоки с использованием OpenCV:

```python
import rospy
import cv2
from sensor_msgs.msg import Image
from cv_bridge import CvBridge

rospy.init_node('computer_vision_sample')
bridge = CvBridge()

def image_callback(data):
    cv_image = bridge.imgmsg_to_cv2(data, 'bgr8')  # OpenCV image
    # Do any image processing with cv2...

image_sub = rospy.Subscriber('main_camera/image_raw', Image, image_callback)

rospy.spin()
```

Для отладки обработки изображения можно публиковать отдельный топик с обработанным изображением:

```python
image_pub = rospy.Publisher('~debug', Image)
```

Публикация обработанного изображения (в конце функции image_callback):

```python
image_pub.publish(bridge.cv2_to_imgmsg(cv_image, 'bgr8'))
```

Получаемые изображения можно просматривать используя [web_video_server](web_video_server.md).

### Примеры

#### Работа с QR-кодами

> **Hint** Для высокоскоростного распознавания и позиционирования лучше использовать [ArUco-маркеры](aruco.md).

Для программирования различных действий коптера при детектировании нужных [QR-кодов](https://ru.wikipedia.org/wiki/QR-код) можно использовать бибилиотеку [ZBar](http://zbar.sourceforge.net). Ее нужно установить в помощью pip:

```bash
sudo pip install zbar
```

Распознавание QR-кодов на Python:

```python
import cv2
import zbar
from cv_bridge import CvBridge
from sensor_msgs.msg import Image

bridge = CvBridge()
scanner = zbar.ImageScanner()
scanner.parse_config('enable')

# Image subscriber callback function
def image_callback(data):
    cv_image = bridge.imgmsg_to_cv2(data, 'bgr8')  # OpenCV image
    gray = cv2.cvtColor(cv_image, cv2.COLOR_BGR2GRAY, dstCn=0)

    pil = ImageZ.fromarray(gray)
    raw = pil.tobytes()

    image = zbar.Image(320, 240, 'Y800', raw)  # Image params
    scanner.scan(image)

    for symbol in image:
        # print detected QR code
        print 'decoded', symbol.type, 'symbol', '"%s"' % symbol.data

image_sub = rospy.Subscriber('main_camera/image_raw', Image, image_callback, queue_size=1)
```

Скрипт будет занимать 100% процессора. Для искусственного замедления работы скрипта можно запустить [throttling](http://wiki.ros.org/topic_tools/throttle) кадров с камеры, например, в 5 Гц (`main_camera.launch`):

```xml
<node pkg="topic_tools" name="cam_throttle" type="throttle"
    args="messages main_camera/image_raw 5.0 main_camera/image_raw/throttled"/>
```

Топик для подписчика в этом случае необходимо поменять на `main_camera/image_raw/throttled`.
