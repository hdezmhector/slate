---
title: Integradores SIFEI

language_tabs: # must be one of https://git.io/vQNgJ
  - php
  - ruby
  - python
  - javascript


toc_footers:
  - <a href='https://www.sifei.com.mx/'>Sifei</a>
  - <a href='#'>Adquiere tus accesos</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - herramientas
  - timbrado
  - soporte
  - glosario

  

search: true
---

# Introducción

Bienvenido a Sifei Docs! Puedes utilizar nuestros servicios para timbrar, cancelar recuperar tus CFDI  además de muchas más otras funciones.

Sifei, es un Proveedor Autorizado de Certificación (PAC) con número de autorización SAT: 58335 que ofrece los servicios de validación, emisión, resguardo, certificación de CFDI y expedición de facturas a través del adquirente de bienes o servicios (PCECFDI).

Contamos con ejemplos en Java, C# ,PHP,Ruby, Python, y JavaScript(nodeJs)! Puedes ver el código de ejemplo en el lado derecho y cambiar al lenguaje de tu interés.

Si necesitas algún ejemplo en algún otro lenguaje favor de solicitarlo vía correo electronico a soporte@sifei.com.mx

## Información Base 
### Genera XML
La estructura para el **XML** versión 3.3, tiene que seguir la "Definición de Esquema XML" ó "XML Schema Definition"(XSD por sus siglas en inglés) este XSD es usado para expresar una serie de reglas a las que un documento XML debe ajustarse para ser considerado como "válido", de acuerdo a ese esquema.

>Documentación técnica - **Anexo 20** *versión 3.3*:

Documento| Fecha Publicación SAT
---------- | ---------- 
[Esquema (xsd) ​](http://www.sat.gob.mx/sitio_internet/cfd/3/cfdv33.xsd "Esquema (xsd)") | 28/07/2017 (Ver. DOF)
[Estándar (pdf)](http://omawww.sat.gob.mx/tramitesyservicios/Paginas/documentos/cfdv33.pdf "Estándar (pdf)") | 28/07/2017 (Ver. DOF)
[Secuencia de Cadena Original (xslt)](http://www.sat.gob.mx/sitio_internet/cfd/3/cadenaoriginal_3_3/cadenaoriginal_3_3.xslt "cadenaoriginal_3_3.xslt") | ​28/09/2018 
[Catálogos CFDI .xsd](http://www.sat.gob.mx/sitio_internet/cfd/catalogos/catCFDI.xsd "Catálogos CFDI.XSD") | 15/01/2020 
[Patrón de datos](http://www.sat.gob.mx/sitio_internet/cfd/tipoDatos/tdCFDI/tdCFDI.xsd "Patrón de Datos.XSD") | 14/12/2017
[Timbre Fiscal Digital 1.1(xsd) ](https://ejemplo.com/ "timbre fiscal digital 1.1") | ​12/04/2017
[Cadena Original del Timbre Fiscal Digital (xslt)​](http://www.sat.gob.mx/sitio_internet/cfd/timbrefiscaldigital/cadenaoriginal_TFD_1_1.xslt "cadena original del timbre fiscal digital (xslt)​") | 29/05/2017
[​Matriz de Errores (xls)](http://omawww.sat.gob.mx/tramitesyservicios/Paginas/documentos/MatrizDeErrores_CFDI_v33.xls "​Matriz de errores (xls)") | 12/04/2017​



### Genera Cadena Original


**Cadena original resultado:**

La cadena original es la secuencia de datos formada con parte de la información contenida dentro del comprobante fiscal digital.

Con la cadena original y tu certificado, podrás generar el sello que también debes colocar en el comprobante.

Para generarla puedes utilizar un transformador xslt o construirla manualmente.

****construcción automática con xslt****
Para la construcción de la cadena original automática, necesitas:

El cfdi con toda la información (ejemplo)
El xslt de transformación
Instalar la herramienta xsltproc (Incluida en Linux y OS X. Para windows descargarla aquí).
Desde la linea de comando deberás ejecutar:

>xsltproc cadenaoriginal_3_2.xslt cfd_ejemplo.xml


**Recibirás la cadena original:**

||3.3|DEMO|28|2020-05-18T10:21:43|01|20001000000300022762|3000|MXN|1|3360|I|PUE|72540|TCM970625MB1|EMPRESA CFDI DEMO SIFEI|601|XAXX010101000|CLIENTE DE MOSTRADOR|P01|78101802|1|E48|PIEZA|(COMPROBANTE DEMO SIN VALOR FISCAL) FLETE DE CONTENEDOR CAIU9040650|3000|3000.00|3000|002|Tasa|0.160000|480|3000|002|Tasa|0.040000|120|002|120|120|002|Tasa|0.160000|480|480||



construcción manual
Si deseas construirla manualmente, estas son las reglas generales:

1. Ninguno de los atributos que conforman al comprobante fiscal digital deberá contener el caracter | (“pipe”) debido a que este será utilizado como caracter de control en la formación de la cadena original.

2. El inicio de la cadena original se encuentra marcado mediante una secuencia de caracteres || (doble “pipe”).

3. Se expresará únicamente la información del dato sin expresar el atributo al que hace referencia. Esto es, si la serie del comprobante es la “A” solo se expresará |A| y nunca |Serie A|.

4. Cada dato individual se encontrará separado de su dato subsiguiente, en caso de existir, mediante un carácter | (“pipe” sencillo).

5. Los espacios en blanco que se presenten dentro de la cadena original serán tratados de la siguiente manera:
6. Se deberán remplazar todos los tabuladores, retornos de carro y saltos de línea por espacios en blanco.
7. Acto seguido se elimina cualquier caracter en blanco al principio y al final de cada separador | (“pipe” sencillo).
8. Finalmente, toda secuencia de caracteres en blanco intermedias se sustituyen por un único caracter en blanco.
9. Los datos opcionales no expresados, no aparecerán en la cadena original y no tendrán delimitador alguno.
10. El final de la cadena original será expresado mediante una cadena de caracteres || (doble “pipe”).
11. Toda la cadena original se expresará en el formato de codificación UTF-8.
12. El nodo o nodos adicionales se integrarán a la cadena original como se indica en la secuencia de formación en su numeral 10, respetando la secuencia de formación y número de orden del ComplemetoConcepto.
13. El nodo o nodos adicionales se integraran al final de la cadena original respetando la secuencia de formación para cada complemento y número de orden del Complemento.

**secuencia de formación**

La secuencia de formación será siempre en el orden que se expresa a continuación, tomando en cuenta las reglas generales expresadas en el párrafo anterior.

**||version|fecha|tipoDeComprobante|formaDePago|condicionesDePago|subTotal|descuento|TipoCambio|Moneda|total|metodoDePago|LugarExpedicion|NumCtaPago|FolioFiscalOrig|SerieFolioFiscalOrig|FechaFolioFiscalOrig|MontoFolioFiscalOrig|Emisor:rfc|Emisor:nombre|Emisor:DomicilioFiscal:calle|Emisor:DomicilioFiscal:noExterior|Emisor:DomicilioFiscal:noInterior|Emisor:DomicilioFiscal:colonia|Emisor:DomicilioFiscal:localidad|Emisor:DomicilioFiscal:referencia|Emisor:DomicilioFiscal:municipio|Emisor:DomicilioFiscal:estado|Emisor:DomicilioFiscal:referencia|Emisor:DomicilioFiscal:estado|Emisor:DomicilioFiscal:pais|Emisor:DomicilioFiscal:codigoPostal|Emisor:ExpedidoEn:calle|Emisor:ExpedidoEn:noExterior|Emisor:ExpedidoEn:noInterior|Emisor:ExpedidoEn:colonia|Emisor:ExpedidoEn:localidad|Emisor:ExpedidoEn:referencia|Emisor:ExpedidoEn:municipio|Emisor:ExpedidoEn:estado|Emisor:ExpedidoEn:referencia|Emisor:ExpedidoEn:estado|Emisor:ExpedidoEn:pais|Emisor:ExpedidoEn:codigoPostal|Emisor:RegimenFiscal:Regimen|Receptor:rfc|Receptor:nombre|Receptor:DomicilioFiscal:calle|Receptor:DomicilioFiscal:noExterior|Receptor:DomicilioFiscal:noInterior|Receptor:DomicilioFiscal:colonia|Receptor:DomicilioFiscal:localidad|Receptor:DomicilioFiscal:referencia|Receptor:DomicilioFiscal:municipio|Receptor:DomicilioFiscal:estado|Receptor:DomicilioFiscal:referencia|Receptor:DomicilioFiscal:estado|Receptor:DomicilioFiscal:pais|Receptor:DomicilioFiscal:codigoPostal|Concepto:cantidad|Concepto:unidad|Concepto:noIdentificacion|Concepto:descripcion|Concepto:valorUnitario|Concepto:importe|Concepto:InformacionAduanera:numero|Concepto:InformacionAduanera:fecha|Concepto:InformacionAduanera:aduana|Concepto:CuentaPredial:numero|ComplementoConcepto|Impuestos:Retencion:impuesto|Impuestos:Retencion:importe|Impuestos:totalImpuestosRetenidos|Impuestos:Traslado:impuesto|Impuestos:Traslado:importe|Impuestos:totalImpuestosTrasladados||
* Los campos no obligatorios en la definición del CFDI, se podrían ignorar en la cadena. Como Emisor:ExpedidoEn Receptor:ExpedidoEn

* Para cada uno de los nodos de Emisor:RegimenFiscal, se deberá incluir cada valor Regimen

* Para cada uno de los nodos de Concepto:InformacionAduanera deberá incluir cada valor numero, fecha, aduana

* Para todos los nodos de Impuestos:Trasladado e Impuestos:Retencion impuesto, importe

Puedes consultar el detalle en el Anexo 20, página 48

### Sellado XML

### Genera Base64

### Genera archivos .PEM

### Genera CFDI Retenciones 3.3

### Genera CFDI Sector Primario 3.3

### Certificados




# Autenticacion


Para autenticar es necesario que adquieras tus credenciales de ambiente de pruebas.

Aqui hay que describir en una tabla o lista el nombre de usuario, pass y id de equipo

Ademas del proceso para obtener estas  HOLA



<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>
