#parte 1
```
import cv2 as cv
import numpy as np 
import os 
```

## Se define el directorio donde se encuentran las carpetas con las imágenes de las emociones, despues definirlo se enlistan las carpetas encontradas y se crean las etiquetas correspondientes
```
dataSet = 'C:\\Users\\zahid\\Documents\\practica1\\emociones'
faces = os.listdir(dataSet)
print(faces)
labels = []
facesData = []
label = 0 
```

##Recorre cada carpeta y cada archivo en el directorio de emociones ademas acada archivo le agrega un uno a la etiqueta
```
for face in faces:
    facePath = dataSet + '\\' + face
    
    for faceName in os.listdir(facePath):
        # Añade la etiqueta actual a la lista de etiquetas
        labels.append(label)
        
        facesData.append(cv.imread(facePath + '\\' + faceName, 0))
    label = label + 1
```
##Imprime el número de imágenes correspondientes a la primera etiqueta (emoción)
`print(np.count_nonzero(np.array(labels) == 0)) `

##Crea un reconocedor de caras usando el algoritmo LBPH 
```
faceRecognizer = cv.face.LBPHFaceRecognizer_create()
faceRecognizer.train(facesData, np.array(labels))
faceRecognizer.write('LBHFace.xml')
```
#parte2
import cv2 as cv
import os
import numpy as np

# en esta parte se carga el archivo xml que se creo anteriormente
```
faceRecognizer = cv.face.LBPHFaceRecognizer_create()
faceRecognizer.read('LBHFace.xml')  
```

# Cargar el clasificador Haar para detectar rostros
`rostro = cv.CascadeClassifier('haarcascade_frontalface_alt2.xml')`

# Iniciar la captura de video
`cap = cv.VideoCapture(1)`

# Lista para almacenar los nombres de las emociones, Obtiene una lista de las carpetas en dataSet
```
dataSet = 'C:\\Users\\zahid\\Documents\\practica1\\emociones\\'
faces = os.listdir(dataSet)
```
#este bloque es un ciclo que sirve para capturar los cuadros del video
```
while True:
    ret, frame = cap.read()
    if not ret:
        break
    ```
#convierte en escala de grises
    ```
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    rostros = rostro.detectMultiScale(gray, 1.3, 3)
     ```
#en este codigo cuando se detecta un rostro se crea un cuadro en el area de intere usando el modelo para poder predecir.
   ```
    for (x, y, w, h) in rostros:
        frame2 = gray[y:y+h, x:x+w]
        frame2 = cv.resize(frame2, (100, 100), interpolation=cv.INTER_CUBIC)
        result = faceRecognizer.predict(frame2)
        ```
#muetrsa los resultados los condicionales sirven para poner el texto de la emocion encontrada y la ultima parte sirve para mostrarlo en una nueva ventana
```
        if result[1] < 2000:  # Ajusta este umbral según sea necesario
            cv.putText(frame, '{}'.format(faces[result[0]]), (x, y-25), 2, 1.1, (0, 255, 0), 1, cv.LINE_AA)
            cv.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
        else:
            cv.putText(frame, 'Desconocido', (x, y-20), 2, 0.8, (0, 0, 255), 1, cv.LINE_AA)
            cv.rectangle(frame, (x, y), (x+w, y+h), (0, 0, 255), 2)
    
    cv.imshow('frame', frame)
    k = cv.waitKey(1)
    if k == 27:
        break
```
cap.release()
cv.destroyAllWindows()







