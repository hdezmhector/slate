
# Timbrado 

## Timbrado(getCFDI):  (SOAP)

```ruby
require 'json'
require 'savon'
require 'iniparse'
require 'time'
require "base64"
require 'zip'

config=IniParse.parse( File.read('config.ini') )

print config
print "hel"

usuario=config['timbrado']['UsuarioSIFEI']
password=config['timbrado']['PasswordSIFEI']
idEquipo=config['timbrado']['IdEquipoGenerado']
usuario=config['timbrado']['UsuarioSIFEI']

#file=File.open('xml_sellado.xml','r');
#cfdi_contenido=file.read
cfdi_contenido=File.read('assets/xml_sellado.xml')

#savon 2 no ofrece una manera directa de acceder al request por lo que deberas de utilizar algun interceptor
client = Savon.client(wsdl: 'http://devcfdi.sifei.com.mx:8080/SIFEI33/SIFEI?wsdl',
    #log: true
    convert_request_keys_to: :none,
    open_timeout: 300,
    read_timeout: 300,
    )
#print client.operations
begin
    #Establecemos el hash(dictionario) con los datos a utilizar!
    soapParams= { 
        Usuario: usuario,
        Password:password ,
        IdEquipo: idEquipo,
        archivoXMLZip: Base64.strict_encode64(cfdi_contenido), # 
        Serie:"" ,
    }
    
    #print soapParams
    #simulamos request para guardarlo y ver como se envia(omitir en produccion)
    operation=client.operation(:get_cfdi)
    request= operation.build(message:soapParams).pretty
    timestamp=Time.now.to_i.to_s    
    File.open("timbrado_request_#{timestamp}.xml" ,"w"){ |file|file.puts request}
    response = client.call(:get_cfdi, message:soapParams)
    # print response.to_xml
    File.open("timbrado_response_#{timestamp}.xml" ,"w"){ |file|file.puts response.to_xml}
    puts "Escribiendo zip"
    zipName="zip_timbrado_#{timestamp}.zip"
    #se decodifica el resultado de la respuesta(base64) y se escribe a un archivo zip que contiene el xml timbrado
    File.open(zipName ,"w"){ |file|file.puts Base64.strict_decode64(response.body[:get_cfdi_response][:return])}


    #una vez escrito el zip, debemos extraer el XML timbrado
    Zip::File.open(zipName) do |zipfile|
        zipfile.each do |file|
          # do something with file
          file.extract
        end
      end
rescue Savon::SOAPFault => e  
    #print client.request
    puts "Exception:"
    puts e.message    
    puts e.http.code    
    puts e.to_hash#[':fault']#[':faultcode']    
    File.open("timbrado_response_#{timestamp}.xml" ,"w"){ |file|file.puts e.http.body}

   # print e.backtrace
    
    #print e.as_json
end
```

```python
 #Ejemplo de ws de timbrado getCFDI(). Para timbrado masivo(con miles de conceptos enviar a )
#Para usar este ejemplo debes instalar Zeep mediante pip
#Comando
#>pip install zeep

from zeep import Client #En este caso de utiliza la libreria Zeep para clientes SOAP. Puede utilizar la libreria de su eleccion
from zeep.exceptions import Fault
from zeep.plugins import HistoryPlugin # con este plugin puedes obtener el historico de peticiones y respuestas

import io
import configparser
from os import path
import os
import time;
from lxml import etree
from zipfile import ZipFile #necesario para extraer el zip donde viene el XML timbrado

#Para evitar colocar accesos directamente en codigo se lee de un archivo.
if not path.exists('config.ini'):
    print ("Archivo de configuracion no existe")
else:      
    config = configparser.ConfigParser()
    config.read('config.ini')

    usuario = config['timbrado']['UsuarioSIFEI']#"Usuario SIFEI"
    password =config['timbrado']['PasswordSIFEI']# "Password SIFEI"
    idEquipo =config['timbrado']['IdEquipoGenerado']# "ID de Equipo SIFEI"

    #directorio relativo a este script compatible con windows y linux(para luego generar el path para leer el XML sellado)
    xmlDIR=os.path.dirname(os.path.abspath(__file__))
    workDir=os.path.dirname(os.path.abspath(__file__))
    timestamp=str(time.time())
    try:
        #Para este ejmplo se envia a timbrar un XML ya sellado  desde una ruta:
        xml_path = os.path.join(xmlDIR,"assets/CFDI33_sellado.xml")
        zipPath = os.path.join(xmlDIR,"assets/CFDI33_timbrado.zip")
        destino_path = os.path.join(xmlDIR,"assets/")
        
        serie = "" #Serie vacia para usuarios de timbrado    
        
        #Se lee el archivo xml establecido en la variable xml_path
        file = open(xml_path, 'r')
        readxml = file.read()
        file.close()
        
        #Se transforma el xml en array de bytes
        xml_bytesArray = bytearray(readxml,'utf-8')
        #Se crea history para cliente SOAP
        history = HistoryPlugin()
        #Se establece el cliente SOAP mediante la clase 'Cliente' de Zeep
        client = Client(
            wsdl="http://devcfdi.sifei.com.mx:8080/SIFEI33/SIFEI?wsdl",
            plugins=[history]  #Plugin para guardar el request y response para la fase de intregracion(podras ver alli los errores arrojados)
        )
        #Se pasan los parametros requeridos para el metodo de timbrado getCFDI()
        result = client.service.getCFDI(usuario, password, xml_bytesArray, serie, idEquipo)
        #Se recibe el xml en caso de ser timbrado(viene dentro de un zip)
        if result != None:
            
            f = open(zipPath, 'wb')
            writer = io.BufferedWriter(f)
            writer.write(result)
            writer.close()
            #una vez escrito el zip, debemos extraer el XML timbrado
            with ZipFile(zipPath, 'r') as zipObj:
                zipObj.extractall(destino_path)

            print("Comprobante almacenado en su sistema.")

    except Fault as fault:
        #Se extrae el request en caso de excepcion
        detail_decoded = client.wsdl.types.deserialize(fault.detail[0])
    #    timbrado = detail_decoded.xml        #Para extraer xml timbrado en caso de utilizar el metodo getCFDISign()
        print(detail_decoded)
    finally:
         #En ambiente de pruebas mandamos el requets y response  a un archivo respecticamente para inspeccionarlos en caso de error, se asigna un timestamp para identificarlos:
        with open(os.path.join(workDir,"timbrado_request_"+timestamp+".xml"),'w', encoding="utf-8") as request:
            request.write(etree.tostring(history.last_sent["envelope"], encoding="unicode", pretty_print=True))
        with open(os.path.join(workDir,"timbrado_response_"+timestamp+".xml"),'w', encoding="utf-8") as request:
            request.write(etree.tostring(history.last_received["envelope"], encoding="unicode", pretty_print=True)) 
 
        
    
```

```php
<?php
/**
 * Ejemplo de timbrado PHP del WS SOAP getCFDI() de SIFEI. * 
 * 
 * El CFDI(XML) debe estar sellado correctamente
 * Nota: Para simplificar los ejemplos todas las rutas son relativas y los datos se leen de un archivo config.ini, lo cual no debe de hacerse en un ambiente de produccion.
 * 
 */
$cfdiSelladoPath=__DIR__."/assets/cfdi.xml";//Ruta del xml sellado
$originalName=basename($cfdiSelladoPath);
#Se lee el contenido del XML
$xml = file_get_contents($cfdiSelladoPath);
$tmpDirName = "tmp"; //nombre del directorio temporal
//Para el atributo archivoXMLZip no es necesario aplicar el base64_encode ya que la misma libreria SoapClient lo hace, por eso la mandamos tal cual.

$array_ini =parse_ini_file(__DIR__."/config.ini");
$parametros=array(
	//accesos
	'Usuario'=>$array_ini['UsuarioSIFEI'], 
	'Password'=>$array_ini['PasswordSIFEI'],
	'IdEquipo'=>$array_ini['IdEquipoGenerado'],
	//archivo XML
 	'archivoXMLZip'=>$xml, 	
 	'Serie'=>''
  	
);

//url de pruebas
$wsdl ="http://devcfdi.sifei.com.mx:8080/SIFEI33/SIFEI?wsdl";
$client = new SoapClient($wsdl, 
	array(
		'soap_version' => SOAP_1_1,
		'trace' => true,
	)
); 

try {
	$res = $client->__soapCall('getCFDI', array($parametros ));
	$fileTmpZip = "timbrado.zip";  //nombre del zip
	//mandamos en un zip el xml timbrado en caso de exito
	file_put_contents( $fileTmpZip, $res->return);
	$zipXml = new ZipArchive();//En caso de not found ZipArchive, asegurate de tener instalada la extension

	if ($zipXml->open($fileTmpZip) === TRUE) {	  		
  		$zipXml->extractTo( $tmpDirName );
		$zipXml->close();
	}
	 
	
} catch (SoapFault $e) {
	#En caso de un error inspeccionar la excepcion:
	var_dump( $e->faultcode, $e->faultstring, $e->detail );
} finally{
	$time=time();
	#En ambiente de pruebas mandamos el requets y response  a un archivo respecticamente para inspeccionarlos en caso de error, se asigna un timestamp para identificarlos:
	//mandamos en un XML el mensaje soap del request
 
	
	$doc=new DOMDocument();
	$doc->loadXML($client->__getLastRequest());
	$doc->formatOutput=true;
	$doc->save(__DIR__."/workfiles/timbrado_getCFDI_request_{$time}.xml");

	//mandamos en un XML el response del xml timbrado
	
	$doc=new DOMDocument();
	$doc->loadXML($client->__getLastResponse());
	$doc->formatOutput=true;
	$doc->save(__DIR__."/workfiles/timbrado_getCFDI_response_{$time}.xml");
}
?>
```
```javascript
/**
 * Ejemplo de timbrado PHP del WS SOAP getCFDI() de SIFEI. * 
 * 
 * El CFDI(XML) debe estar sellado correctamente
 * Nota: Para simplificar los ejemplos todas las rutas son relativas y los datos se leen de un archivo config.ini, lo cual no debe de hacerse en un ambiente de produccion.
 * Los datos de pruebas y produccion son distintos, deberas solicitar primero tus datos de pruebas y una vez finalizadas tus pruebas deberas reemplazar tus accesos y URL de Ws service.
 */
var soap = require('soap');
var fs = require('fs');
var ini = require('ini')
var unzipper= require('unzipper')
var path = require('path');


//cargamos los accesos desde un almacenamiento seguro, en este caso para facilitar el ejemplo se lee desde un archivo .ini
var config=ini.parse(fs.readFileSync('./config.ini','utf-8'));
var usuario=config.timbrado.UsuarioSIFEI;
var password=config.timbrado.PasswordSIFEI;
var idEquipo=config.timbrado.IdEquipoGenerado;



//leemos el XML desde un archivo local,nota, la library propuesta no detecta que tipo requerido para archivoXMLZip es un binario y no lo codifica a base 64 
//por lo que se debe de realizar
cfdi= fs.readFileSync('./assets/cfdi.xml','utf-8');
let cfdiBase64=( Buffer.from(cfdi,'utf-8')).toString('base64');
//console.log(cfdi)
//URL de pruebas
var url = 'http://devcfdi.sifei.com.mx:8080/SIFEI33/SIFEI?wsdl';

//preparamos los parametros de timbrado
var parametrosDeTimbrado = {
    Usuario: usuario,   //usuario de sifei
    Password:password , //contraseña
    IdEquipo: idEquipo, //id de equipo
    archivoXMLZip: cfdiBase64, //archivo CFDI
    Serie:"" ,
};
soap.createClient(url, function(err, client) {
   
   console.log(JSON.stringify(client.describe()))//con esta linea averguaguamos los metodos
    //invamos metodo de timbrado:
   client.getCFDI(parametrosDeTimbrado,function(err,res,rawSoapResponse,soapResponseHeader,rawSoapRequest){
        //#En ambiente de pruebas mandamos el requets y response  a un archivo respecticamente para inspeccionarlos en caso de error, se asigna un timestamp para identificarlos:
        let timestamp=(new Date()).getTime()
        
        fs.writeFileSync(`./tmp/timbrado_response_${timestamp}.xml`,rawSoapResponse)
        fs.writeFileSync(`./tmp/timbrado_request_${timestamp}.xml`,rawSoapRequest)

        //ahora continuamos con el flujo normal
        if(err){
           //ocurrio un error , se debe leer la excepcion de sifei para ver cual es el error en el XML            
           console.error("error")
           //mensaje de error
           console.error(err.message)
           //excepcion completa(es un JSON tranformado):
            /**
             * {
                    codigo: 'XX',
                    error: 'XXXX',
                    message: 'XXXX'
                }
             */
           console.error(err.root.Envelope.Body.Fault.detail.SifeiException)
        }else{
            console.debug("ok")
           //Podemos extraer el cfdi timbrado  (viene dentro de un zip) 
          // console.log(res)
            let zipP='./tmp/timbrado.zip';
            let timbradoDir=path.dirname(zipP).split(path.sep).pop()
            //la respuesta vieene en el return
            fs.writeFileSync(zipP,res.return,{
              encoding:'base64'  
            })  
            //ahora debemos leer el zip y extraer los archivos(solo es el XML timbrado)
            fs.createReadStream(zipP)
                .pipe(unzipper.Extract({
                    path:timbradoDir
                }))
            console.log("Archivos extraidos")
        }
   })
});
```

> Solictud generada:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tns="http://MApeados/" xmlns:env="http://schemas.xmlsoap.org/soap/envelope/">
  <env:Body>
    <tns:getCFDI>
      <Usuario>TCM....</Usuario>
      <Password>123456..</Password>
      <IdEquipo>123dsa..</IdEquipo>
      <archivoXMLZip>PD94bWwgd..</archivoXMLZip>
      <Serie/>
    </tns:getCFDI>
  </env:Body>
</env:Envelope>

```
> Respuesta retornada:

```xml
<?xml version='1.0' encoding='UTF-8'?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body>
<ns2:getCFDIResponse xmlns:ns2="http://MApeados/">
<return>UEsDBBQACAAIAI5...</return>
</ns2:getCFDIResponse></S:Body>
</S:Envelope>
```

Este metodo se encarga de timbrar un CFDI, por tanto es el metodo mas importante.

<p class="light"></p>
```xml
<?xml version='1.0' encoding='UTF-8'?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body>
<ns2:getCFDIResponse xmlns:ns2="http://MApeados/">
<return>UEsDBBQACAAIAI5...</return>
</ns2:getCFDIResponse></S:Body>
</S:Envelope>
```


### URL de timbrado
La siguiente es la URL de pruebas, con ella podras consumir y acceder a todos los metodos disponibles


`http://devcfdi.sifei.com.mx:8080/SIFEI33/SIFEI`


### Parametros de la solicitud

A continuacion se muestran los parametros necesarios para consumir el ws

Parametro | Tipo | Descripcion
--------- | ------- | -----------
Usuario | String | Usuario sifei
Password | String | If setset to false, the result will include kittens that have already been adopted.
IdEquipo | String | If setset to false, the result will include kittens that have already been adopted.
archivoXMLZip | Binario(base64) | XML a timbrar
Serie | Binario(base64) | XML a timbrar


<aside class="success">
Recuerda solicitar tus accesos para poder realizar tus pruebas!
</aside>

<aside class="notice">
 Anteriormente el servicio requiería que el XML estuviera dentro de un archivo zip
</aside>

### Excepción

 Cuando se produce un error en el servicio la respuesta SOAP genera una excepcion llamada SifeiException:


<p class="light"></p>
```xml
<?xml version='1.0' encoding='UTF-8'?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
<S:Body>
<S:Fault xmlns:ns4="http://www.w3.org/2003/05/soap-envelope">
<faultcode>S:Server</faultcode>
<faultstring>Parámetros incompletos</faultstring>
<detail>
<ns2:SifeiException xmlns:ns2="http://service.sifei.cancelacion/">
<codigo>1003</codigo>
<detalle>Parámetros incompletos</detalle>
<error>Parámetros incompletos</error>
</ns2:SifeiException>
</detail>
</S:Fault>
</S:Body>
</S:Envelope>
```

### Codigos de error

Es importante mencionar ...

Codigo de error| Significado
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your API key is wrong.
403 | Forbidden -- The kitten requested is hidden for administrators only.
404 | Not Found -- The specified kitten could not be found.
405 | Method Not Allowed -- You tried to access a kitten with an invalid method.
406 | Not Acceptable -- You requested a format that isn't json.
410 | Gone -- The kitten requested has been removed from our servers.
418 | I'm a teapot.
429 | Too Many Requests -- You're requesting too many kittens! Slow down!
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.


## Timbrado (getCFDIProcesa):  (SOAP)

Recibir un cfdi para procesarlo posteriormente, respondiendo con un mensaje el cual indica que se recibió correctamente, así como el id de seguimiento, este CFDI será timbrado en el caso de haber sido correcto y para obtener el timbre se deberá consumir el método getXMLProceso.


<p class="light"></p>
```xml
<?xml version='1.0' encoding='UTF-8'?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
   <S:Body>
      <ns2:getCFDIProcesaResponse xmlns:ns2="http://MApeados/">
         <return>Cfdi en proceso de validacion, verificar mas tarde</return>
         <return>62227649</return>
      </ns2:getCFDIProcesaResponse>
   </S:Body>
</S:Envelope>
```

### URL de timbrado
La siguiente es la URL de pruebas, con ella podras consumir y acceder a todos los metodos disponibles


`http://devcfdi.sifei.com.mx:8080/SIFEI33/SIFEI`


### Parametros de la solicitud

A continuacion se muestran los parametros necesarios para consumir el ws

Parametro | Tipo | Descripcion
--------- | ------- | -----------
Usuario | String | Usuario sifei
Password | String | If setset to false, the result will include kittens that have already been adopted.
IdEquipo | String | If setset to false, the result will include kittens that have already been adopted.
archivoXMLZip | Binario(base64) | XML a timbrar
Serie | Binario(base64) | XML a timbrar


<aside class="success">
Recuerda solicitar tus accesos para poder realizar tus pruebas!
</aside>

<aside class="notice">
 Anteriormente el servicio requiería que el XML estuviera dentro de un archivo zip
</aside>

### Excepción

 Cuando se produce un error en el servicio la respuesta SOAP genera una excepcion llamada SifeiException:


<p class="light"></p>
```xml
<?xml version='1.0' encoding='UTF-8'?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
<S:Body>
<S:Fault xmlns:ns4="http://www.w3.org/2003/05/soap-envelope">
<faultcode>S:Server</faultcode>
<faultstring>Parámetros incompletos</faultstring>
<detail>
<ns2:SifeiException xmlns:ns2="http://service.sifei.cancelacion/">
<codigo>1003</codigo>
<detalle>Parámetros incompletos</detalle>
<error>Parámetros incompletos</error>
</ns2:SifeiException>
</detail>
</S:Fault>
</S:Body>
</S:Envelope>
```

### Codigos de error

Es importante mencionar ...

Codigo de error| Significado
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your API key is wrong.
403 | Forbidden -- The kitten requested is hidden for administrators only.
404 | Not Found -- The specified kitten could not be found.
405 | Method Not Allowed -- You tried to access a kitten with an invalid method.
406 | Not Acceptable -- You requested a format that isn't json.
410 | Gone -- The kitten requested has been removed from our servers.
418 | I'm a teapot.
429 | Too Many Requests -- You're requesting too many kittens! Slow down!
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.

 