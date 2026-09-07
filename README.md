# Digitalizador de caracterizaciones

Herramienta interna de la Subred Integrada de Servicios de Salud Sur E.S.E. para
digitar los consolidados de caracterización recolectados en visitas de campo.

## Qué hace

Convierte un formato escaneado en un archivo de Excel, guiando la digitación
celda por celda. El navegador se encarga de lo mecánico —enderezar la hoja,
detectar la cuadrícula, recortar y ampliar cada casilla— y la persona solo
teclea lo que ve.

No hace reconocimiento automático de escritura. A la resolución habitual de
estos escaneos ningún motor de OCR alcanza la precisión que exige un número de
documento, así que la lectura sigue siendo humana y lo que se automatiza es
todo lo demás.

## Poblaciones

- Adultos
- Gestantes (semanas de gestación, controles, FUM)
- Menores de 5 años (peso, talla, datos de quien recibe la visita)
- Recién nacidos (peso y talla al nacer, edad gestacional)

## Cómo se usa

1. Elige la población y arrastra el PDF o la imagen.
2. Gira la hoja si viene de lado y usa el enderezado automático.
3. Detecta la cuadrícula. Ajusta la sensibilidad si encuentra líneas de más o de menos.
4. Digita. `Enter` avanza, `Shift+Enter` retrocede, `Ctrl+I` marca la celda como ilegible.
5. Descarga el Excel.

El archivo exportado incluye una columna `REVISAR` con los campos que quedaron
dudosos o ilegibles, y una hoja de control con el conteo. Esos campos deben
contrastarse contra el documento físico antes de cargarlos a la base.

## Validaciones

El número de documento se valida según el tipo declarado en la misma fila:
registro civil entre 10 y 11 dígitos, cédula entre 6 y 10. El teléfono acepta
fijo de 7 o celular de 10. Las fechas se normalizan a `AAAA-MM-DD`. Peso, talla
y semanas de gestación se validan contra rangos fisiológicos.

## Tratamiento de datos

Todo el procesamiento ocurre dentro del navegador. El archivo cargado no se
sube a ningún servidor y no queda almacenado al cerrar la pestaña. El botón de
guardar avance usa `sessionStorage`, que se borra al cerrar el navegador.

Los datos tratados son datos personales y algunos son sensibles. Aplican la Ley
1581 de 2012 y el Decreto 1377 de 2013. No publiques capturas de pantalla con
información de identificación en el repositorio ni en incidencias.

## Agregar una población

En `index.html`, dentro del objeto `PLANTILLAS`, agrega una entrada nueva
partiendo de `BASE` y sumando las columnas propias:

```js
MI_POBLACION: {
  nombre: "Nombre visible",
  columnas: [...BASE.map(c => ({...c})),
    {k:"MI_CAMPO", t:"entero", ancho:.8, num:true, min:0, max:100, opcional:true}
  ]
}
```

Tipos disponibles: `texto`, `nombre`, `cc`, `rc`, `doc`, `tel`, `fecha`,
`entero`, `decimal`, `lista`. El motor no requiere cambios.

## Dependencias

PDF.js y SheetJS desde CDN. Sin build, sin backend, sin instalación.

## Estado

En validación. La detección de cuadrícula depende de la calidad del escaneo;
funciona mejor a 300 DPI con la hoja alineada.
