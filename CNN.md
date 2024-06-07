# PARTE 1
## esta parte del codigo sirve para importar las imagenes del database a utilizar en este caso para desastres, robo, robo a casa, inundacion, incendio, tornado, las cuales se van ir almacenando en una lista
```
import os
import numpy as np
import matplotlib.pyplot as plt
import re  # Importar el módulo de expresiones regulares

dirname = os.path.join(os.getcwd(),'C:\\Users\\zahid\\Documents\\practica 4\\datasetDesastres\\')
imgpath = dirname + os.sep 

images = []
directories = []
dircount = []
prevRoot=''
cant=0

print("leyendo imagenes de ",imgpath)
```
## en esta parte se utilizan ciclos for con la finalidad de ir recorriendo cada imagen y verificando si su extesion pertenece a las especificadas para asi ir guardandolas
```
for root, dirnames, filenames in os.walk(imgpath):
    for filename in filenames:
        if re.search("\.(jpg|jpeg|png|bmp|tiff)$", filename):
            cant=cant+1
            filepath = os.path.join(root, filename)
            image = plt.imread(filepath)
            if(len(image.shape)==3):
                
                images.append(image)
            b = "Leyendo..." + str(cant)
            print (b, end="\r")
            if prevRoot !=root:
                print(root, cant)
                prevRoot=root
                directories.append(root)
                dircount.append(cant)
                cant=0
dircount.append(cant)
```
## en esta parte se crea un contador e imprime el numero de directorios asi como el numero de imagenes y el total por todas las imagenes existentes
```
dircount = dircount[1:]
dircount[0]=dircount[0]+1
print('Directorios leidos:',len(directories))
print("Imagenes en cada directorio", dircount)
print('suma Total de imagenes en subdirs:',sum(dircount))
```
## en esta parte se crean etiquetas por el total de imagenes creadas
```
labels=[]
indice=0
for cantidad in dircount:
    for i in range(cantidad):
        labels.append(indice)
    indice=indice+1
print("Cantidad etiquetas creadas: ",len(labels))
```
## en esta parte se va creando una lista de directorios ademas se les va agregando un indece a cada inicializando en 0 y sumando 1 cada directorio nuevo
```
tragedias = []
indice = 0
for directorio in directories:
    # Extrae el nombre del directorio final del path
    name = directorio.split(os.sep)[-1]
    print(indice, name)
    tragedias.append(name) 
    indice += 1
```


## esta parte convierte la lista labels a un numpy array. Esto es útil para realizar operaciones numéricas y manipulaciones más eficientes que con listas normales de Python, ademas verifica las clases unicas encontradas e imprime el resultado.
```
y = np.array(labels)
X = np.array(images, dtype=np.uint8) 
classes = np.unique(y)
nClasses = len(classes)
print('Total number of outputs : ', nClasses)
print('Output classes : ', classes)
```

## Se dividen los datos en conjuntos en este caso se utilizo un %20 para prueba y un %80 para entrenamiento, utilizando train_test_split. Se imprimen las formas de los conjuntos de entrenamiento y prueba.
```
from sklearn.model_selection import train_test_split
train_X,test_X,train_Y,test_Y = train_test_split(X,y,test_size=0.2)
print('Training data shape : ', train_X.shape, train_Y.shape)
print('Testing data shape : ', test_X.shape, test_Y.shape)
```

## ets parte tiene la finalidad de mostrar ejemplos de las imaganes cargadas para el entrenamineto
```
plt.figure(figsize=[5,5])
plt.subplot(121)
plt.imshow(train_X[0,:,:], cmap='gray')
plt.title("Ground Truth : {}".format(train_Y[0]))
plt.subplot(122)
plt.imshow(test_X[0,:,:], cmap='gray')
plt.title("Ground Truth : {}".format(test_Y[0]))
```

## Tiene la funcionalidad onvertir las imágenes a tipo de dato float32 esto con el fin de realizar operaciones aritméticas precisas y para preparar los datos para modelos de aprendizaje automático que suelen requerir entradas en formato de punto flotante. una vez hecho la coneversion muestra una visualización de imágenes.
```
train_X = train_X.astype('float32')
test_X = test_X.astype('float32')
train_X = train_X/255.
test_X = test_X/255.
plt.imshow(test_X[0,:,:])
```
## Cambia las etiquetas de codificación categórica a one-hot
```
train_Y_one_hot = to_categorical(train_Y)
test_Y_one_hot = to_categorical(test_Y)
```

## Muestra el cambio para la etiqueta de categoría usando codificación one-hot
```
print('Original label:', train_Y[0])
print('After conversion to one-hot:', train_Y_one_hot[0])
```
# Mezcla todo y crea los grupos de entrenamiento y testing
```
train_X,valid_X,train_label,valid_label = train_test_split(train_X, train_Y_one_hot, test_size=0.2, random_state=13)

print(train_X.shape,valid_X.shape,train_label.shape,valid_label.shape)
```
# esta parte tiene como finalidad crear un modelo CNN donde en las primeras lineas se establecen los parametros de entrenamiento e interacciones, ademas se define las dimensiones del kernel asi como la forma de las entradas, asi como en la parte final se definen las capas
```
INIT_LR = 1e-3  # Valor inicial del learning rate
epochs = 35  # Número de épocas
batch_size = 32  # Tamaño del lote
nClasses = 5  # Asumiendo que tienes 5 clases diferentes de tragedias

sport_model = Sequential()
sport_model.add(Conv2D(32, kernel_size=(3, 3), activation='linear', padding='same', input_shape=(30, 30, 3)))
sport_model.add(LeakyReLU(alpha=0.1))
sport_model.add(MaxPooling2D((2, 2), padding='same'))
sport_model.add(Dropout(0.5))

sport_model.add(Flatten())
sport_model.add(Dense(32, activation='linear'))
sport_model.add(LeakyReLU(alpha=0.1))
sport_model.add(Dropout(0.5))
sport_model.add(Dense(nClasses, activation='softmax'))

sport_model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['accuracy'])
```
## se muestra el resultado de lo creado anteriormente
sport_model.summary()

## se mide el rendimiento
```
sport_model.compile(
    loss=keras.losses.categorical_crossentropy, 
    optimizer=tf.keras.optimizers.SGD(learning_rate=INIT_LR, decay=INIT_LR / 100),
    metrics=['accuracy']
)
```
## Este código entrena el modelo sportmodel utilizando los datos de entrenamiento, ajustando los parámetros del modelo para minimizar la función loss definida, ademas evalúa el rendimiento del modelo en los datos.
```
sport_train = sport_model.fit(train_X, train_label, batch_size=batch_size,epochs=epochs,verbose=1,validation_data=(valid_X, valid_label))
```
## esta lineas sirven para guardar el archivo del modelo de entrenamiento ademas tiene una verificacion si el directorio existe si no existe este se creara
```
ruta = "C:\\Users\\zahid\\Documents\\practica 4\\datasetsTrag\\CNN\\tragedias.h5"
if not os.path.exists(os.path.dirname(ruta)):
    print('Carpeta creada: ', os.path.dirname(ruta))
    os.makedirs(os.path.dirname(ruta))
sport_model.save(ruta)
```
## en esta parte  se hace un test o una evaluacion del modelo
`test_eval = sport_model.evaluate(test_X, test_Y_one_hot, verbose=1)`

## se imprime la presicion del modelo
```
print('Test loss:', test_eval[0])
print('Test accuracy:', test_eval[1])
```
## este codigo sirve para hacer un analisis de los datos de presicion y perdida asi mismo graficas cada uno
```
accuracy = sport_train.history['accuracy']
val_accuracy = sport_train.history['val_accuracy']
loss = sport_train.history['loss']
val_loss = sport_train.history['val_loss']
epochs = range(len(accuracy))
plt.plot(epochs, accuracy, 'bo', label='Training accuracy')
plt.plot(epochs, val_accuracy, 'b', label='Validation accuracy')
plt.title('Training and validation accuracy')
plt.legend()
plt.figure()
plt.plot(epochs, loss, 'bo', label='Training loss')
plt.plot(epochs, val_loss, 'b', label='Validation loss')
plt.title('Training and validation loss')
plt.legend()
plt.show()
```
## Utiliza el modelo entrenado para predecir las etiquetas de las imágenes en el conjunto de datos de prueba y Almacena las predicciones del modelo
`predicted_classes2 = sport_model.predict(test_X)`

## Este código realiza un proceso manual para convertir las probabilidades de predicción (predicted_classes2) en etiquetas de clase
```
predicted_classes=[]
for predicted_sport in predicted_classes2:
    predicted_classes.append(predicted_sport.tolist().index(max(predicted_sport)))
predicted_classes=np.array(predicted_classes)
predicted_classes.shape, test_Y.shape
```

## este codigo encuentra los índices de las muestras en las que las predicciones del modelo coinciden con las etiquetas verdaderas una vez encontrados los indices se almacenan y se imprime la cantidad de muestras correctamente clasificadas.
```
correct = np.where(predicted_classes == test_Y)[0]
print("Found %d correct labels" % len(correct))
for i, correct in enumerate(correct[0:9]):
    plt.subplot(3,3,i+1)
    plt.imshow(test_X[correct], cmap='gray', interpolation='none')  
    plt.title("{}, {}".format(tragedias[predicted_classes[correct]], tragedias[test_Y[correct]]))
    plt.tight_layout()
```
## parecido a lo anterior pero ahora con muestras incorrectas e imprime lo incorrecto
 ```
    incorrect = np.where(predicted_classes != test_Y)[0]
print("Found %d incorrect labels" % len(incorrect))
for i, incorrect in enumerate(incorrect[0:9]):
    plt.subplot(3,3,i+1)
    plt.imshow(test_X[incorrect], cmap='gray', interpolation='none')  # Asumiendo que test_X[incorrect] ya tiene el tamaño correcto
    plt.title("{}, {}".format(tragedias[predicted_classes[incorrect]], tragedias[test_Y[incorrect]]))
    plt.tight_layout()
 ```
## esta parte calcula la presicion del modelo dependiendo cada clase encontrada
 ```
nClasses = len(np.unique(test_Y)) 
target_names = ["Class {}".format(i) for i in range(nClasses)]
print(classification_report(test_Y, predicted_classes, target_names=target_names))
 ```
## esta parte sirve para imprimir los parametros de entrenamiento asi como las perdidas y las precision del modelo.
```
print(f"INIT_LR = {INIT_LR}\nepochs = {epochs}\nbatch_size = {batch_size}\n")
print('Test loss:', test_eval[0])
print('Test accuracy:', test_eval[1])
```
## impirme las etiquetas correctas y las incorrectas
```
correct = np.where(predicted_classes==test_Y)[0]
print("Found %d correct labels" % len(correct))
incorrect = np.where(predicted_classes!=test_Y)[0]
print("Found %d incorrect labels" % len(incorrect))
```
## Carga del modelo
```
model = load_model("C:\\Users\\zahid\\Documents\\practica 4\\datasetsTrag\\CNN\\tragedias.h5")
```
## Procesamiento de la imagen, se redimensiona la imagen a 30x30 píxeles para que coincida con el tamaño de entrada esperado por el model asi como los otros ajustes necesarios para el correcto funcionamiento
```
def preprocess_image(image_path):
    img = Image.open(image_path)
    img = img.resize((30, 30))  
    img_array = np.array(img)
    img_array = np.expand_dims(img_array, axis=0)
    return img_array
```
## Ruta de las imagenes que deseas probar
```
dataTest = "C:\\Users\\zahid\\Documents\\practica 4\\datasetsTrag\\pruebaCNN"
imgs  = os.listdir(dataTest)
```
## primero se construye la ruta completa de la imagen. despues se procesa la imagen utilizando la función definida anteriormente. y al ultimo realiza una predicción sobre la imagen preprocesada insertando el nombre del archivo y la etiqueta de la clase predicha a la lista results.
```
results = []

for img in imgs:
    imgpath = dataTest+'/'+img
    # Preprocesamiento de la imagen
    image = preprocess_image(imgpath)
    # Realizar predicciones
    predictions = model.predict(image)
    
    results.append([img, tragedias[np.argmax(predictions)]])
```
## estas lineas de codigo tiene como funcion mostrar de una forma visible lo anterior descrito o bueno el resultado de lo anterior
```
plt.figure(figsize=(15, 15))

num_images = len(results)

for i, result in enumerate(results):
    img_path = os.path.join(dataTest, result[0])
    img = Image.open(img_path)
    plt.subplot((num_images // 3) + 1, 3, i + 1)
    plt.imshow(img)
    plt.title(f'Predicción: {result[1]}\n')
    plt.axis('off')
    plt.text(0, -5, result[0] + '\n', ha='left')

plt.tight_layout()
plt.show()
```