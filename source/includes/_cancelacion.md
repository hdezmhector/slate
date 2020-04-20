
# Cancelacion

## Cancelar CFDI (cancelaCFDI):

Contenido

```php
<?php
/**
* Ejemplo de cancelacion WS SOAP cancelaCFDI()
* Nota: Para simplificar los ejemplos todas las rutas son relativas y los datos se leen de un archivo, lo cual no debe de hacerse en un ambiente de produccion.
* # PFX CONFORMADO POR key y cert
*/
$array_ini =parse_ini_file(__DIR__."/config.ini");

#Parametros requeridos para consumir el metodo de cancelación
$usuarioSIFEI = $array_ini['UsuarioSIFEI'];//;"usuario SIFEI";
$passwordSifei =$array_ini['PasswordSIFEI']; //"password SIFEI";

$rfcEmisor = "RFC del emisor";
$pfx = file_get_contents(__DIR__."/".$array_ini["PFX"]); #se lee el contenido del PFX, solo como ejemplo se lee de un directorio
$passwordPfx ="a0123456789"; #"contraseña del pfx";
$uuids[]= "uuid";
//Para el atributo pfx no es necesario aplicar el base64_encode ya que SoapClient se encarga de realizarlo.
$parametros = array(
	'usuarioSIFEI'=>$usuarioSIFEI,
 	'passwordSifei'=>$passwordSifei,
  	'rfcEmisor'=>$rfcEmisor, 
  	'pfx'=>$pfx, 
  	'passwordPfx'=>$passwordPfx,
  	"uuids"=>$uuids
);
#print_r($parametros);
//url de pruebas
$wsdl ="http://devcfdi.sifei.com.mx:8888/CancelacionSIFEI/Cancelacion?wsdl";

$client = new SoapClient($wsdl, 
				array(  
				'soap_version' => SOAP_1_1,
				'trace' => true,
		)); 


try {
	$res = $client->__soapCall( 'cancelaCFDI', array($parametros ));
	
	file_put_contents("acuse.xml", $res->return );
	echo "Se guardo acuse";
} catch (SoapFault $e) { 
	#obtenemos la excepcion:
	var_dump( $e->faultcode, $e->faultstring, $e->detail );
} finally{
	$time=time();
	#En ambiente de pruebas mandamos el requets y response  a un archivo respecticamente para inspeccionarlos en caso de error, se asigna un timestamp para identificarlos:
	//mandamos en un XML el mensaje soap del request
	file_put_contents("cancelaCFDI_request_{$time}.xml", $client->__getLastRequest());
	//mandamos en un XML el response del xml timbrado
	file_put_contents("cancelaCFDI_response_{$time}.xml", $client->__getLastResponse());	
}
?>
```

```ruby
#Requiere isntalar la gema savon

#gem install savon
#gem install iniparse
require 'json'
require 'savon'
require 'iniparse'
require 'time'
require "base64"

config=IniParse.parse( File.read('config.ini') )

print config
print "hel"

usuario=config['timbrado']['UsuarioSIFEI']
password=config['timbrado']['PasswordSIFEI']
idEquipo=config['timbrado']['IdEquipoGenerado']
pfxPath=config['timbrado']['PFX']

#file=File.open('xml_sellado.xml','r');
#cfdi_contenido=file.read
pfx=File.read(pfxPath)

#savon 2 no ofrece una manera directa de acceder al request por lo que deberas de utilizar algun interceptor
client = Savon.client(wsdl: 'http://devcfdi.sifei.com.mx:8888/CancelacionSIFEI/Cancelacion?wsdl',
    #log: true
    )
    passwordPfx ="a0123456789"
#print client.operations
begin
    #Establecemos el hash(dictionario) con los datos a utilizar!
    soapParams= { 
        usuarioSIFEI: usuario,
        passwordSifei:password ,
        rfcEmisor: 'RFC',
        pfx: Base64.encode64(pfx), # PFX CONFORMADO POR key y cert
        passwordPfx: passwordPfx, #         
        uuids:'uuids'
    }
    
    print soapParams
    #simulamos 
    operation=client.operation(:cancela_cfdi)
    request= operation.build(message:soapParams).pretty
    timestamp=Time.now.to_i.to_s    
    File.open("cancelacion_request_#{timestamp}.xml" ,"w"){ |file|file.puts request}
    response = client.call(:cancela_cfdi, message:soapParams)
    # print response.to_xml
    File.open("cancelacion_response_#{timestamp}.xml" ,"w"){ |file|file.puts response.to_xml}
    puts "Acuse"
    #puts response.body
    puts response.body[:cancela_cfdi_response][:return];
    File.open("cancelacion_acuse_#{timestamp}.xml" ,"w"){ |file|file.puts response.body[:cancela_cfdi_response][:return]}

rescue Savon::SOAPFault => e  
    #print client.request
    puts "Exception:"
    puts e.message    
    puts e.http.code    
    puts e.to_hash#[':fault']#[':faultcode']    
    File.open("cancelacion_response_#{timestamp}.xml" ,"w"){ |file|file.puts e.http.body}

   # print e.backtrace
    
    #print e.as_json
end
 
```

## getCFDIProcesa

contenido




## codigos de error
