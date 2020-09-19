# Writeup Challenge2 Onapsis Lockdown Games
>Author: Tomàs Tropea

# Analisis & Resoluciòn

Tenemos 4 archivos en este challenge. Hay dos que se llaman **Server.jar** y **Client.jar**, que son los ejecutables que se comunican entre ellos. Luego los otros dos son archivos donde se hace un registro de la comunicacion que hubo, y se llaman **cripto.Key.log** y **communication.dispatchers.ServerDispatcher.log**. 
En el **communication.dispatchers.ServerDispatcher.log** podemos ver contenido de la forma:

```
Agent 1943329308 registered with key uid: 0.37766619797099044 
Secret agent 1943329308 login succesful. 
New message from Agent 1943329308. 
New response to Agent 1943329308. 
New message from Agent 1943329308. 
New response to Agent 1943329308. 
Secret agent 1943329308 has set a new secret key. 
New message from Agent 1943329308. 
New response to Agent 1943329308.   
Flag sent to secret agent 1943329308. 
```
Pareceria mostrar el tipo de mensaje que se estuvo enviando. Entre los tipos de mensaje, vemos algunos que indican como registrarse inicialmente, entrar en modo **Secret agent**, cambiar la Key para encriptar mensajes y enviar la flag. Por el otro lado, en el **cripto.Key.log** podemos ver:

```
New message from Agent 1943329308. Ecnrypted message: wqZUwpECEsOAw5nCm1E= 
New message to Agent 1943329308. Ecnrypted message: wq5Zwp8DH8OCw5LCn1nCncK1R8Km 
New message from Agent 1943329308. Ecnrypted message: NE5BCgPDqWJsMW1vwq9wwr1ZKSvCqcKOS8Odw6DDsDNhMA== 
New message to Agent 1943329308. Ecnrypted message: BgIGC0HCgQQxZCc+w6gYwo5cPznCvMKDD8KAwo3DuDdrLgYKCQ5HwoU= 
New message from Agent 1943329308. Ecnrypted message: BlxUAQXDik9kO2x5wrJAwoNGOCrCv8KXVsOHw5vCpHIhKRVPTwEUw5NVeix+f8KvQMKfUSA5wqrCiFbDlw== 
New message to Agent 1943329308. Ecnrypted message: ClJCHBjDmV93On58wr5RwopKPz3CsMKfVcOQw5jCqXU= 
New message from Agent 1943329308. Ecnrypted message: FV5AGhTDhUVgPHh0wq9Owp9fPyzCoMKNUMOKw4s= 
New message to Agent 1943329308. Ecnrypted message: E1paCg/Dkl5+PGF+wrpdwopdLivCosKfSsOXw57CqmQuMgtUWgYNw4hFYTdhYcKyQcKKRCQ/wrLCgF3DkMOewrVuIA== 
New message from Agent 1943329308. Ecnrypted message: KcO6YsOmwqU3w4fCj8Onw6HCusKew5HDpcKOGBZ3w544AcKZVTsXwpQ= 
New message to Agent 1943329308. Ecnrypted message:  
New message from Agent 1943329308. Ecnrypted message: PMKkIsO8wrHDssKCw6/Dm8KZw442wp9uw5M8JsOGSnLDnVjDvVQjwrc8wqwyw73CvcOuwpg= 
New message to Agent 1943329308. Ecnrypted message: IsKuMMOiwrDDrsKBw6jDi8KXw4M7woxtw5cmN8OXXnXDhkPDol83wr88wqQ2w6fCqMO/wo/DqcOAwoDDhyzCkmTDiCwrw4xbf8OMTsOxRCPCtC/Cpi/Dt8Kzw67CmcOiw4vCksOFJsKV 
New message from Agent 1943329308. Ecnrypted message: OsKpNsO3wqfDpMKVw7TDkMKKw44jwpJpw5E8J8OaQU7DtmfDr0c= 
New message to Agent 1943329308. Ecnrypted message: OsKpNsO3wqfDpMKVw7TDkMKKw44jwpJpw5E8J8OaQW7DlkfDr1IkwrslwqQhw7rCqMOtwpnDsMOQwoLDgiTCkGzDmS89w5Q= 
```
Cada uno de estos mensajes se corresponde con cada tipo de mensaje enviado, indicado por el archivo anterior. Entonces, una posible forma de resolver este challenge es tratar de desencriptar los mensajes, para poder llegar hasta el mensaje que se corresponde con:
> Flag sent to secret agent 1943329308

Y potencialmente descubrir la flag del challenge. 

Lo primero que note luego de examinar el codigo, es que la *Key* se utilizaba para encriptar y desencriptar mensajes. Como al final voy a querer saber que dicen los mensajes, veamos primero como funciona la *Key*. Si examinamos el **Client.jar** o el **Server.jar** (que utilizan la misma logica de comunicacion), vemos que las keys hacen uso de estas funciones:

```java
public Key() {
    this(System.currentTimeMillis());
  }
  
  public Key(String key) {
    String extKey = String.valueOf(key) + "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    this.key = extKey.substring(0, 26);
    this.uid = -1.0D;
    this.seed = -1L;
  }
  
  public Key(long seed) {
    this.seed = seed;
    Random r = new Random(seed);
    this.uid = r.nextDouble();
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 26; i++)
      sb.append((char)(int)(r.nextDouble() * 256.0D)); 
    this.key = sb.toString();
  }
```

Inicialmente, tanto el cliente como el servidor utilizan una key inicial, que està fija al momento de iniciar la comunicacion. En el codigo a continuacion se puede ver la generacion de esta:


```java
public class Agent implements Serializable {
  private static final long serialVersionUID = -1710769432036898456L;
  
  private int uid;
  
  private transient Key key;
  
  public static final Key SERVER_KEY = new Key(79824387432L);
  
  public Agent() {
    Random r = new Random(System.currentTimeMillis());
    this.uid = r.nextInt();
    this.key = new Key(26L);
  }
  
  public Agent(long seed) {
    Random r = new Random(System.currentTimeMillis());
    this.uid = r.nextInt();
    this.key = new Key(seed);
  }
  
  public int getUid() {
    return this.uid;
  }
  
  protected void setUid(int uid) {
    this.uid = uid;
  }
  
  public Key getKey() {
    return this.key;
  }
  
  public void setKey(Key key) {
    this.key = key;
  }
  
  public boolean equals(Object o) {
    Agent agent = (Agent)o;
    if (agent.getUid() == getUid())
      return true; 
    return false;
  }
  
  public int hashCode() {
    return getUid();
  }
}
```

Si vemos especificamente la linea que dice:
```java
public static final Key SERVER_KEY = new Key(79824387432L);
```
Podremos saber que key utilizan para comunicarse inicialmente. Luego de que el agente se registra en el servidor, este le envia algo al cliente, que veremos a continuacion. Esto aparece en el siguiente codigo:

```java
public boolean Register(Agency agency, HandShake request) throws UnknownHostException, IOException {
    Agent cliAgent = request.getSource();
    if (request.getSeed() == 987654321L) {
      if (agency.addAgent(cliAgent) == 1) {
        Key newKey = new Key();
        HandShake response = new HandShake((Agent)agency, cliAgent, newKey.getSeed());
        cliAgent.setKey(newKey);
        sendMessage((Message)response, Agent.SERVER_KEY);
        Platform.write("Agent " + String.valueOf(cliAgent.getUid()) + " registered.");
        Loger.logInfo(ServerDispatcher.class.getName(), "Agent " + String.valueOf(cliAgent.getUid()) + " registered with key uid: " + String.valueOf(newKey.getUid()));
        return true;
      } 
      sendErrorMsg((ControlMessage)new HandShake((Agent)agency, cliAgent), 2);
      return false;
    } 
    sendErrorMsg((ControlMessage)new HandShake((Agent)agency, cliAgent), 1);
    return false;
  }
```
Lo mas importante de esto son las lineas:
```java
Key newKey = new Key()
HandShake response = new HandShake((Agent)agency, cliAgent, newKey.getSeed());
```
Esta es la nueva clave que se utilizara, pero no se la esta enviando al cliente directamente, sino que envia la *seed* que la genero. De cualquier forma, como ya conocemos el algoritmo para generar la *Key*, basta con saber como se encripta/desencripta un mensaje, y luego armar un script que haga los pasos correctos para desencriptar la flag finalmente.

La parte de encriptar y desencriptar lo podemos observar en el siguiente codigo:

```java
  protected String encrypt(String data) {
    data = data.replace(" ", "~");
    data = data.replace(".", "|");
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < data.length(); i++) {
      int indx = i % 26;
      char enc = (char)(data.charAt(i) ^ this.key.charAt(indx));
      sb.append(enc);
    } 
    return Base64.getEncoder().encodeToString(sb.toString().getBytes());
  }
  
  protected String decrypt(String data) {
    String decData = new String(Base64.getDecoder().decode(data));
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < decData.length(); i++) {
      int indx = i % 26;
      char dec = (char)(decData.charAt(i) ^ this.key.charAt(indx));
      sb.append(dec);
    } 
    String out = sb.toString();
    out = out.replace("~", " ");
    out = out.replace("|", ".");
    return out;
  }
  
  public Message encrypt(Message m) {
    String data = m.getData();
    String ciph = encrypt(data);
    m.setData(ciph);
    return m;
  }
  
  public Message decrypt(Message m) {
    String data = m.getData();
    String ciph = decrypt(data);
    m.setData(ciph);
    return m;
  }
```

Entonces ya tenemos casi todo, sabemos como generar la *Key* inicial para la comunicacion, y tambien como encriptar y desencriptar los mensajes. Lo unico que falta es saber que pasa cuando un agente secreto cambia la *Key*. 
Cuando un agente secreto cambia su clave, llama al constructor *new Key()*, como muestra el **case 1**, y genera una clave utilizando como seed el tiempo actual en milisegundos. Esto no complica la obtencion de esa nueva key, ya que el agente se la envia al server directo, y podemos desencriptar el mensaje con la *Key* actual, y asi obtener la nueva para los siguientes mensajes.

```java
 public void secretActions() throws UnknownHostException, IOException, NoSuchAlgorithmException, ClassNotFoundException {
    int code;
    StringBuilder flag;
    String msg;
    getDispatcher().openConnection(getServer());
    int action = getSecretOption();
    switch (action) {
      case 1:
        getClient().setKey(new Key());
        code = getDispatcher().setAgentKey(getClient(), getServer(), getClient().getKey());
        if (code == 0) {
          Platform.write("Key established. All communication will be encrypted with your own Key.");
          break;
        } 
        writeErrorMessage(code);
        break;
      case 2:
        flag = new StringBuilder();
        code = getDispatcher().retrieveFlag((SecretAgent)getClient(), getServer(), flag);
        if (code == 0) {
          Platform.write(flag.toString());
          break;
        } 
        writeErrorMessage(code);
        break;
      case 3:
        msg = Platform.writeReadMsg();
        msg = getDispatcher().communicate(getClient(), getServer(), msg);
        if (msg != null)
          Platform.write(msg); 
        break;
      case 4:
        code = getDispatcher().CloseConnection(getClient(), getServer());
        if (code == 0) {
          Platform.write("Connection closed. You can register again whenever you want!");
          setRegistered(false);
          break;
        } 
        writeErrorMessage(code);
        break;
      case 5:
        Platform.write("Good Bye!");
        getDispatcher().closeConnection();
        System.exit(0);
        break;
    } 
    getDispatcher().closeConnection();
  }
  
```

Con toda esta informacion, podemos escribir un script que tome los mensajes que aparecen en **cripto.Key.log**, y tambien la informacion de cada tipo de mensaje de **communication.dispatchers.ServerDispatcher.log**, para saber cuando se cambia la *Key* para la comunicacion. Luego, si desencriptamos los mensajes entre un agente y el servidor que hayan expuesto el flag, obtendremos lo que queremos. 

Por ejemplo, podemos utilizar este script, desencriptando los mensajes que se mostraron al principio:

```java
import java.util.Random;
import java.util.Base64;
import java.util.Random;


public class Main
{
	public static final int SIZE = 26;
	
	public static void main(String[] args) {
    
      long seed = 79824387432L;
      String key = Key(seed);   // generamos la key inicial para comunicarse
      //----------------------- Mensajes a desencriptar ----------------------------------
      decrypt("wqZUwpECEsOAw5nCm1E=",key);
      // generamos la nueva key con la seed que nos envio el servidor al registrarnos
      seed = Long.parseLong(decrypt("wq5Zwp8DH8OCw5LCn1nCncK1R8Km",key),10);
      key = Key(seed); 
      //seguimos desencriptando mensajes..
      decrypt("NE5BCgPDqWJsMW1vwq9wwr1ZKSvCqcKOS8Odw6DDsDNhMA==",key);
      decrypt("BgIGC0HCgQQxZCc+w6gYwo5cPznCvMKDD8KAwo3DuDdrLgYKCQ5HwoU=",key);
      decrypt("BlxUAQXDik9kO2x5wrJAwoNGOCrCv8KXVsOHw5vCpHIhKRVPTwEUw5NVeix+f8KvQMKfUSA5wqrCiFbDlw==",key);
      decrypt("ClJCHBjDmV93On58wr5RwopKPz3CsMKfVcOQw5jCqXU=",key);
      decrypt("FV5AGhTDhUVgPHh0wq9Owp9fPyzCoMKNUMOKw4s=",key);
      decrypt("E1paCg/Dkl5+PGF+wrpdwopdLivCosKfSsOXw57CqmQuMgtUWgYNw4hFYTdhYcKyQcKKRCQ/wrLCgF3DkMOewrVuIA==",key);
      //cambiamos la key en el momento que el secret agent la cambia
      key = decrypt("KcO6YsOmwqU3w4fCj8Onw6HCusKew5HDpcKOGBZ3w544AcKZVTsXwpQ=",key);
      //seguimos desencriptando mensajes..
      decrypt("PMKkIsO8wrHDssKCw6/Dm8KZw442wp9uw5M8JsOGSnLDnVjDvVQjwrc8wqwyw73CvcOuwpg=",key);
      decrypt("IsKuMMOiwrDDrsKBw6jDi8KXw4M7woxtw5cmN8OXXnXDhkPDol83wr88wqQ2w6fCqMO/wo/DqcOAwoDDhyzCkmTDiCwrw4xbf8OMTsOxRCPCtC/Cpi/Dt8Kzw67CmcOiw4vCksOFJsKV",key);
      decrypt("OsKpNsO3wqfDpMKVw7TDkMKKw44jwpJpw5E8J8OaQU7DtmfDr0c=",key);
      decrypt("OsKpNsO3wqfDpMKVw7TDkMKKw44jwpJpw5E8J8OaQW7DlkfDr1IkwrslwqQhw7rCqMOtwpnDsMOQwoLDgiTCkGzDmS89w5Q=",key);

	}
	
	public static String Key(long seed){
	    
		Random r = new Random(seed);
		double uid = r.nextDouble();
		StringBuilder sb = new StringBuilder();
		for (int i = 0;i < 26;i ++)
		    sb.append((char)(int)(r.nextDouble() * 256.0D));
		String key = sb.toString();
		
		return key;
	}
	
	public static String decrypt(String data,String key) {
        String decData = new String(Base64.getDecoder().decode(data));
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < decData.length(); i++) {
          int indx = i % 26;
          char dec = (char)(decData.charAt(i) ^ key.charAt(indx));
          sb.append(dec);
        } 
        String out = sb.toString();
        out = out.replace("~", " ");
        out = out.replace("|", ".");
        System.out.println(out);
        return out;
    }
}
```
El resultado de correr este script seràn los siguientes mensajes desencriptados:

```
987654321
1597868690480
Super_Secret_Password_123.
a97d075868437cdeabb692969ba18a63
agent. mission bravo dessert needs autorization
mission have green light
requesting target list
take down targets. snake. loki. the king. gladiator
NÁSÔöµþ°Eþ¶BN©?¸&:EØ
requesting safe house information
lockdown is ready at evergreen. you will need the flag. good luck
the secret flag is ONA{}
the secret flag is ona{hackers.love.randoms}
```
Donde la flag es el ultimo mensaje enviado:

>ona{hackers.love.randoms}
