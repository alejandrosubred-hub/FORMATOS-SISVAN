# Digitalizador de caracterizaciones

Herramienta interna de la Subred Integrada de Servicios de Salud Sur E.S.E. para
convertir los consolidados de caracterización recolectados en visitas de campo
en un archivo de Excel.

Alejandro Ortega — Subred Sur · SISVAN

## Qué hace

Sueltas el formato escaneado y la herramienta lo deja listo para digitar. Por su
cuenta reconoce a qué población corresponde, corrige la orientación de la hoja,
la endereza, detecta la tabla y va mostrando cada casilla ampliada junto a un
campo donde tecleas lo que ves.

La lectura de lo escrito a mano sigue siendo humana. A la resolución habitual de
estos escaneos ningún motor de reconocimiento acierta lo suficiente en un número
de documento, y un dígito equivocado es un registro que nunca cruza con la base.
Lo que se automatiza es todo lo demás: el operador no pelea con una hoja
rotada, no salta renglones y recibe aviso cuando un dato no cuadra.

El encabezado del formato sí es texto impreso, y ahí el reconocimiento
automático funciona bien: de ahí sale la población y, de paso, la rotación
correcta.

## Poblaciones

- Adultos
- Gestantes — semanas de gestación, número de controles, FUM
- Menores de 5 años — peso, talla, datos de quien recibe la visita
- Recién nacidos — peso y talla al nacer

## Cómo se usa

1. Arrastra el PDF o la imagen. El resto ocurre solo.
2. Confirma la población que reconoció. Si se equivocó, corrígela en el
   desplegable.
3. Digita. `Enter` avanza, `Shift+Enter` retrocede, `Ctrl+I` marca la celda como
   ilegible, `Ctrl+↓` salta a la fila siguiente.
4. Descarga el Excel.

Si algo no se detectó bien, el panel **Ajustar la detección** deja cambiar de
página, girar la hoja, corregir la inclinación a mano y mover la sensibilidad de
línea. Se abre solo cuando la detección automática falla.

## El archivo exportado

Una hoja con los datos y una hoja **Control** con el conteo del lote y la
autoría. La columna `REVISAR` lista, fila por fila, los campos que quedaron
dudosos o ilegibles. Esos deben contrastarse contra el documento físico antes de
cargarlos a la base.

El resaltado en ámbar y rojo se ve en la tabla de la herramienta mientras se
digita, no en el archivo descargado: la versión libre de SheetJS escribe datos
pero no colores de celda.

## Validaciones

El número de documento se valida según el tipo declarado en la misma fila:
registro civil entre 10 y 11 dígitos, cédula entre 6 y 10, y cualquier otro tipo
como documento alfanumérico. El teléfono acepta fijo de 7 o celular de 10. Las
fechas se normalizan a `AAAA-MM-DD`. Peso, talla, semanas de gestación y edad
gestacional se contrastan contra rangos fisiológicos.

Nada de esto bloquea la digitación: un valor fuera de rango se guarda igual y
queda marcado para revisión.

## Tratamiento de datos

Todo el procesamiento ocurre dentro del navegador. El archivo cargado no se
sube a ningún servidor y no queda almacenado al cerrar la pestaña.

Los datos tratados son datos personales y algunos son sensibles. Aplican la Ley
1581 de 2012 y el Decreto 1377 de 2013. No publiques capturas de pantalla con
información de identificación en el repositorio ni en incidencias. El
`.gitignore` bloquea PDF, imágenes y hojas de cálculo para que ningún escaneo se
suba por descuido.

## Agregar una población

En `index.html`, dentro del objeto `PLANTILLAS`, agrega una entrada partiendo de
`BASE` y sumando las columnas propias. En `claves` van las palabras que aparecen
en el encabezado impreso del formato, sin tildes y en minúscula, que son las que
usa el reconocimiento para identificarlo:

```js
MI_POBLACION: {
  nombre: "Nombre visible",
  claves: ["palabra del encabezado"],
  columnas: [...clonar(BASE),
    {k:"MI_CAMPO", t:"entero", ancho:.8, num:true, min:0, max:100, opcional:true}
  ]
}
```

Tipos disponibles: `texto`, `nombre`, `cc`, `rc`, `doc`, `tel`, `fecha`,
`entero`, `decimal`, `lista`. El motor no requiere cambios.

## Dependencias

PDF.js, Tesseract.js y SheetJS desde CDN. Sin build, sin backend, sin
instalación. Tesseract descarga el modelo de español la primera vez que se abre
la herramienta en un equipo y lo deja en caché; si la red bloquea el CDN, el
reconocimiento del encabezado falla sin romper nada y solo hay que elegir la
población a mano.

## Sobre llevarlo a una base de datos

El navegador no puede escribir en MySQL sin un backend, y montar uno implica
servidor, credenciales y obligaciones adicionales de habeas data. La ruta
sensata es en dos tiempos: la herramienta exporta el Excel o el CSV, y un script
local lo carga a la base con control de duplicados sobre `(tipo_id, numero_id)`
y registro del lote. Así la parte web sigue sin infraestructura y la base queda
dentro de la red institucional.

## Estado

En validación. La detección de la tabla depende de la calidad del escaneo;
funciona mejor a 300 DPI, en escala de grises y con la hoja alineada.
