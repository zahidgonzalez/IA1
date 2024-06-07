## parte del codigo donde se importan las librerias necesarias
```
import cv2 as cv
import numpy as np
import pickle
import os
```
## se carga el modelo svm
```
with open('wally_svm_model2.pkl', 'rb') as f:
    svm_model = pickle.load(f)
```

## Este codigo toma una imagen en color y la convierte a escala de grises. Luego, redimensiona la imagen a un tamaño específicado. Calcula el descriptor HOG para la imagen redimensionada y lo devuelve como un vector unidimensional.
```
def extract_features(img, size=(64, 128)):
    gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
    resized = cv.resize(gray, size)
    hog = cv.HOGDescriptor()
    h = hog.compute(resized)
    return h.flatten()
```
## Se carga la imagen en imagen. windowsize especifica el tamaño de la ventana (x, y) que se deslizará sobre la imagen para detectar a "Wally". stepsize indica del deslizamiento de la ventana.
```
image_path = 'C:/users/zahid/Documents/practica3/Buscar/7.png'
image = cv.imread(image_path)
window_size = (28, 48)  
step_size = 50
found = False
detections = []
```

## se hace un dezplasamiento en el vector x y y atravez de los ciclos for, mediante ek svm model se hace una prediccion y se va guardando lo detectado
```
for y in range(0, image.shape[0] - window_size[1], step_size):
    for x in range(0, image.shape[1] - window_size[0], step_size):
        sub_img = image[y:y + window_size[1], x:x + window_size[0]]
        features = extract_features(sub_img)
        prediction = svm_model.predict([features])
        if prediction == 1:  # Wally encontrado
            detections.append((x, y, window_size[0], window_size[1]))
```
## elimina lo que no se necesita, mediante una interaccion de cajas, donde atraves de analizis de coordenas se van determinando las no necesarias y regresa solo las necesarias
```
def non_max_suppression(boxes, overlapThresh):
    if len(boxes) == 0:
        return []

    boxes = np.array(boxes)
    pick = []

    x1 = boxes[:, 0]
    y1 = boxes[:, 1]
    x2 = boxes[:, 0] + boxes[:, 2]
    y2 = boxes[:, 1] + boxes[:, 3]

    area = (x2 - x1 + 1) * (y2 - y1 + 1)
    idxs = np.argsort(y2)

    while len(idxs) > 0:
        last = len(idxs) - 1
        i = idxs[last]
        pick.append(i)

        xx1 = np.maximum(x1[i], x1[idxs[:last]])
        yy1 = np.maximum(y1[i], y1[idxs[:last]])
        xx2 = np.minimum(x2[i], x2[idxs[:last]])
        yy2 = np.minimum(y2[i], y2[idxs[:last]])

        w = np.maximum(0, xx1 - xx2 + 1)
        h = np.maximum(0, yy1 - yy2 + 1)

        overlap = (w * h) / area[idxs[:last]]

        idxs = np.delete(idxs, np.concatenate(([last], np.where(overlap > overlapThresh)[0])))

    return boxes[pick].astype("int")
```
## Se aplica la supresión para eliminar áreas superpuestas. Se dibuja un rectángulo verde alrededor de cada detección y se agrega un texto "Aqui esta". se establece en True si se encontró al menos una detección.
```
detections = non_max_suppression(detections, overlapThresh=0.3)

for (x, y, w, h) in detections:
    cv.rectangle(image, (x, y), (x + w, y + h), (0, 255, 0), 2)
    cv.putText(image, 'Aqui esta', (x, y - 10), cv.FONT_HERSHEY_SIMPLEX, 0.9, (0, 255, 0), 2)
    found = True
```
## Si no se encontraron detecciones, se imprime un mensaje indicando que "Wally" no fue encontrado
```
if not found:
    print("Wally no encontrado en la imagen.")
```
## Se ajusta el tamaño de la ventana de visualización para mostrar la imagen resultante con las detecciones dibujadas.
```
height, width, _ = image.shape

resize_factor = 0.7  
new_width = int(width * resize_factor)
new_height = int(height * resize_factor)
```
## Finalmente, se muestra la imagen en una ventana con OpenCV.
```
cv.namedWindow('Donde esta Wally', cv.WINDOW_NORMAL)
cv.resizeWindow('Donde esta Wally', new_width, new_height)
cv.imshow('Donde esta Wally', image)
cv.waitKey(0)
cv.destroyAllWindows()
```