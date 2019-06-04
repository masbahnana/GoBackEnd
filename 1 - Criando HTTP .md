# Criando o primeiro server 
O Go foi criado para resolver os problemas que surgiram com a nova arquitetura de processadores de vários núcleos, criando redes de alto desempenho que atendem a milhões de solicitações e trabalhos que exigem muitos cálculos. A ideia por trás do Go era aumentar a produtividade, permitindo a criação rápida de protótipos, diminuindo a compilação e o tempo de construção, e permitindo um melhor gerenciamento de dependências.

Ao contrário da maioria das outras linguagens de programação, o Go fornece o pacote net / http, que é suficiente ao criar clientes e servidores HTTP. Este capítulo abordará a criação de servidores HTTP e TCP no Go.

Começaremos com algumas receitas simples para criar um servidor HTTP e TCP e passaremos gradualmente para receitas mais complexas, nas quais implementamos a autenticação básica, otimizamos as respostas do servidor, definimos várias rotas e registramos solicitações HTTP. Também abordaremos conceitos e palavras-chave, como Go Handlers, Goroutines e Gorilla - um kit de ferramentas da web para o Go.

## Criando um simples HTTP server
vamos criar um servidor HTTP simples que renderize o Hello World! quando navegamos ```http: // localhost: 8080``` ou executamos o curl ```http: // localhost: 8080``` a partir da linha de comando. Siga os passos:

- 1.Na pasta que você criou para a aula crie um arquivo chamado ```http-server.go``` e faça como está abaixo:

```
package main
import 
(
  "fmt"
  "log"
  "net/http"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
func helloWorld(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Hello World!")
}
func main() 
{
  http.HandleFunc("/", helloWorld)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```



- 2.No terminal vá até a pasta onde está o seu código e faça como abaixo:

```$ go run http-server.go```


### Como isso deve funcionar:
Uma vez executado o programa, um servidor HTTP iniciará escutando localmente na porta ```8080```. Abrindo ```http: // localhost: 8080``` em um navegador exibirá Hello World!

####Entenda o que cada linha significa:
- ```package main```: Isto define o nome do pacote do programa.

- ```import ("fmt" "log" "net / http")```: Este é um comando pré-processador que informa ao compilador Go para incluir todos os arquivos do ```fmt```, ```log``` e o pacote ```net / http```.

- ```const (CONN_HOST = "localhost" CONN_PORT = "8080")```: Declaramos constantes no programa Go usando a palavra-chave ```const```. Aqui declaramos duas constantes - uma é ```CONN_HOST``` com localhost como um valor e outra é ```CONN_PORT``` com ```8080``` como um valor.

- ```func helloWorld (w http.ResponseWriter, r * http.Request) {fmt.Fprintf (w, "Hello World!")}```: Esta é uma função Go que toma ```ResponseWriter``` e Request como uma entrada e escreve ```Hello World!``` em um fluxo de resposta HTTP.

Em seguida, declaramos o método main () de onde a execução do programa começa, já que esse método faz muitas coisas. Vamos entender linha por linha:

- ```http.HandleFunc ("/", helloWorld)```: Aqui, estamos registrando a função ```helloWorld``` com o padrão / URL usando HandleFunc do pacote ```net / http```, o que significa que helloWorld é executado, passando ```(http.ResponseWriter, * http.Request)``` como um parâmetro para ele sempre que acessar o URL HTTP com padrão /.

- ```err: = http.ListenAndServe (CONN_HOST + ":" + CONN_PORT, nil)```: Aqui, estamos chamando ```http.ListenAndServe``` para atender a solicitações HTTP que tratam cada conexão de entrada em um Goroutine separado. O ```ListenAndServe``` aceita dois parâmetros - endereço do servidor e manipulador. Aqui, estamos passando o endereço do servidor como ```localhost: 8080``` e handler como ```nil```, o que significa que estamos solicitando ao servidor que use o ```DefaultServeMux``` como um manipulador.

- ```if err! = nil {log.Fatal ("erro ao iniciar servidor http:", err) return}```: Aqui, verificamos se existe um problema ao iniciar o servidor. Se houver, registre o erro e saia com um código de status ```1```.

## Implementando autenticação básica em um servidor HTTP simples
Depois de criar o servidor HTTP, você provavelmente desejará restringir recursos de serem acessados por um usuário específico, como o administrador de um aplicativo. Em caso afirmativo, você pode implementar a autenticação básica em um servidor HTTP

Como já criamos um servidor HTTP em nosso passo a passo anterior, vamos apenas estendê-lo para incorporar a autenticação básica.

- 1.Vamos atualizar o servidor HTTP que criamos no passo anterior, adicionando uma função BasicAuth e modificando o HandleFunc para chamá-lo.
Na pasta que você criou para a aula crie um arquivo chamado ```http-server-basic-authentication.go``` e faça como está abaixo:

```
package main
import 
(
  "crypto/subtle"
  "fmt"
  "log"
  "net/http"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
  ADMIN_USER = "admin"
  ADMIN_PASSWORD = "admin"
)
func helloWorld(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Hello World!")
}
func BasicAuth(handler http.HandlerFunc, realm string) http.HandlerFunc {
  return func(w http.ResponseWriter, r *http.Request) 
  {
    user, pass, ok := r.BasicAuth()
    if !ok || subtle.ConstantTimeCompare([]byte(user),
    []byte(ADMIN_USER)) != 1||subtle.ConstantTimeCompare([]byte(pass), 
    []byte(ADMIN_PASSWORD)) != 1 
    {
      w.Header().Set("WWW-Authenticate", `Basic realm="`+realm+`"`)
      w.WriteHeader(401)
      w.Write([]byte("You are Unauthorized to access the
      application.\n"))
      return
    }
    handler(w, r)
  }
}
func main() 
{
  http.HandleFunc("/", BasicAuth(helloWorld, "Please enter your
  username and password"))
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 2.No terminal vá até a pasta onde está o seu código e faça como abaixo:

```$ go run http-server-basic-authentication.go``` 

### Como isso deve funcionar:
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

Quando o servidor for iniciado, o acesso a ```http: // localhost: 8080``` em um navegador solicitará que você insira um nome de usuário e uma senha. Fornecendo-o como ```admin, admin```, respectivamente, renderizará Hello World! na tela, e para qualquer outra combinação de nome de usuário e senha, você não poderá acessar o aplicativo.

Para acessar o servidor a partir da linha de comando, temos que fornecer o sinalizador ```--user``` como parte do comando curl, da seguinte maneira:

```
$ curl --user admin:admin http://localhost:8080/
Hello World!
```

Também podemos acessar o servidor usando um token codificado base64 do nome de ```usuário: senha``` ou ```username: password```, que podemos obter de qualquer site, como ```https://www.base64encode.org/```, e passá-lo como um cabeçalho de autorização no comando curl, como:

```
$ curl -i -H 'Authorization:Basic YWRtaW46YWRtaW4=' http://localhost:8080/

HTTP/1.1 200 OK
Date: Sat, 03 Jun 2019 12:02:51 GMT
Content-Length: 12
Content-Type: text/plain; charset=utf-8
Hello World!
```

####Entenda o que cada linha significa:
- A função ```import``` adiciona um pacote adicional, ```crypto / subtle```, que usaremos para comparar o nome de usuário e a senha das credenciais inseridas pelo usuário.

- Usando a função ```const```, definimos duas constantes adicionais, ```ADMIN_USER``` e ```ADMIN_PASSWORD```, que serão utilizadas durante a autenticação do usuário.

- Em seguida, declaramos um método ```BasicAuth ()```, que aceita dois parâmetros de entrada - um manipulador, que é executado após a autenticação bem-sucedida do usuário, e o realm, que retorna o ```HandlerFunc```, da seguinte maneira:

```
func BasicAuth(handler http.HandlerFunc, realm string) http.HandlerFunc 
{
  return func(w http.ResponseWriter, r *http.Request)
  {
    user, pass, ok := r.BasicAuth()
    if !ok || subtle.ConstantTimeCompare([]byte(user),
    []byte(ADMIN_USER)) != 1||subtle.ConstantTimeCompare
    ([]byte(pass),
    []byte(ADMIN_PASSWORD)) != 1
    {
      w.Header().Set("WWW-Authenticate", `Basic realm="`+realm+`"`)
      w.WriteHeader(401)
      w.Write([]byte("Unauthorized.\n"))
      return
    }
    handler(w, r)
  }
}
```

No anterior anterior, obtemos primeiro o nome de usuário e a senha fornecidos no cabeçalho de autorização da solicitação usando ```r.BasicAuth ()``` e comparamos com as constantes declaradas no programa. Se as credenciais corresponderem, ele retornará o manipulador, caso contrário, ele definirá ```WWW-Authenticate``` juntamente com um código de status ```401``` e gravará ```Você não está autorizado a acessar o aplicativo``` (ou ``` You are Unauthorized to access the application```) em um fluxo de resposta HTTP.

Finalmente, introduzimos uma mudança no método ```main ()``` para chamar ```BasicAuth``` do ```HandleFunc```, da seguinte maneira:

```
http.HandleFunc("/", BasicAuth(helloWorld, "Please enter your username and password"))
```

Acabamos de passar um manipulador ```BasicAuth``` em vez de ```nil``` ou ```DefaultServeMux``` para manipular todas as solicitações recebidas com o padrão de URL como /.

## Otimizando respostas do servidor HTTP com compactação GZIP (responses server HTTP)
A compactação GZIP significa enviar a resposta ao cliente do servidor em um formato ```.gzip```, em vez de enviar uma resposta simples, e é sempre uma boa prática enviar respostas compactadas se um cliente / navegador oferecer suporte a ela.

Ao enviar uma resposta comprimida, economizamos largura de banda e tempo de download, tornando a página mais rápida. O que acontece na compactação GZIP é que o navegador envia um cabeçalho de solicitação informando ao servidor que aceita conteúdo compactado ```(.gzip e .deflate)``` e, se o servidor tiver a capacidade de enviar a resposta em formato compactado, a envia. Se o servidor suportar compactação, ele definirá ```Content-Encoding: gzip``` como cabeçalho de resposta, caso contrário, enviará uma resposta simples ao cliente, o que significa claramente que uma resposta compactada é apenas uma solicitação do navegador e não uma demanda. Nós usaremos o pacote de manipuladores do Gorilla para implementá-lo neste passo.

Vamos criar um servidor HTTP com um único manipulador, que irá escrever Hello World! em um fluxo de resposta HTTP e use um Gorilla ```CompressHandler``` para enviar todas as respostas de volta ao cliente no formato ```.gzip```. Execute os seguintes passos:

- 1.Para usar os manipuladores do Gorilla, primeiro precisamos instalar o pacote usando o comando ```go get``` ou copiá-lo manualmente para ```$ GOPATH / src ou $ GOPATH```, da seguinte forma:

```
$ go get github.com/gorilla/handlers
```

- 2.Crie ```http-server-mux.go``` e siga o código abaixo: 

```
package main
import 
(
  "io"
  "net/http"
  "github.com/gorilla/handlers"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
func helloWorld(w http.ResponseWriter, r *http.Request) 
{
  io.WriteString(w, "Hello World!")
}
func main() 
{
  mux := http.NewServeMux()
  mux.HandleFunc("/", helloWorld)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT,
  handlers.CompressHandler(mux))
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.No terminal vá até a pasta onde está o seu código e faça como abaixo:

```$ go run http-server-mux.go```

### Como isso deve funcionar:
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

Abrindo ```http: // localhost: 8080``` em um navegador exibirá Hello World! do servidor com o valor do cabeçalho de resposta Content-Encoding gzip.

## Criando um simples TCP Server
Sempre que você precisa construir sistemas orientados para alto desempenho, escrever um servidor TCP é sempre a melhor opção em relação a um servidor HTTP, já que os soquetes TCP são menos robustos que o HTTP. O Go suporta e fornece uma maneira conveniente de escrever servidores TCP usando um pacote de rede.

Nesse passo, vamos criar um servidor TCP simples que aceitará uma conexão em ```localhost: 8080```. Execute os seguintes passos:

- 1.Crie ```tcp-server.go``` e copie o seguinte código:

```
package main
import 
(
  "log"
  "net"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
  CONN_TYPE = "tcp"
)
func main() 
{
  listener, err := net.Listen(CONN_TYPE, CONN_HOST+":"+CONN_PORT)
  if err != nil 
  {
    log.Fatal("Error starting tcp server : ", err)
  }
  defer listener.Close()
  log.Println("Listening on " + CONN_HOST + ":" + CONN_PORT)
  for 
  {
    conn, err := listener.Accept()
    if err != nil 
    {
      log.Fatal("Error accepting: ", err.Error())
    }
    log.Println(conn)
  }
}
```

- 2.Depois rode o seguinte comando no terminal:

```$ go run tcp-server.go```

### Como isso deve funcionar:
Assim que executarmos o programa, o servidor TCP começará a escutar localmente na porta ```8080```.

O que cada linha significa:

- ```package main```: Isto define o nome do pacote do programa.

- ```import ("log" "net")```: Este é um comando pré-processador que informa ao compilador Go para incluir todos os arquivos do pacote ```log``` e ```net```.

- ```const (CONN_HOST = "localhost" CONN_PORT = "8080" CONN_TYPE = "tcp")```: Declaramos constantes em um programa Go usando a palavra-chave const. Aqui, declaramos três constantes - uma é ```CONN_HOST``` com um valor de ```localhost```, outra é ```CONN_PORT``` com um valor como ```8080``` e, por último, ```CONN_TYPE``` com um valor como ```tcp```.

Em seguida, declaramos o método main () de onde começa a execução do programa. Como esse método faz muitas coisas, vamos ver linha por linha:

- ```listener, err: = net.Listen (CONN_TYPE, CONN_HOST + ":" + CONN_PORT)```: isso cria um servidor TCP em execução no host local na porta ```8080```.

- ```if err! = nil {log.Fatal ("Erro ao iniciar servidor tcp:", err)}```: Aqui, verificamos se existe algum problema em iniciar o servidor TCP. Se houver, registre o erro e saia com um código de status 1.

- ```defer listener.Close ()```: Essa instrução defere fecha um "ouvinte" de soquete TCP quando o aplicativo é fechado.


Em seguida, aceitamos a solicitação recebida para o servidor TCP em um loop constante e, se houver algum erro na aceitação da solicitação, nós a registramos e saímos; caso contrário, simplesmente imprimimos o objeto de conexão no console do servidor, da seguinte maneira:


## Lendo dados de uma conexão TCP
Um dos cenários mais comuns em qualquer aplicativo é o cliente interagindo com o servidor. O TCP é um dos protocolos mais utilizados para essa interação. O Go fornece uma maneira conveniente de ler dados de conexão de entrada através do bufio implementando Input / Output em buffer, que será abordado nesta receita.

Como já criamos um servidor TCP em nossa receita anterior, vamos atualizá-lo para ler dados de conexões de entrada.

Vamos atualizar o método main () para chamar um método handleRequest passando o objeto de conexão para ler e imprimir dados no console do servidor. Execute os seguintes passos:

- 1.Crie ```tcp-server-read-data.go``` e copie o seguinte código:

```
package main
import 
(
  "bufio"
  "fmt"
  "log"
  "net"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
  CONN_TYPE = "tcp"
)
func main() 
{
  listener, err := net.Listen(CONN_TYPE, CONN_HOST+":"+CONN_PORT)
  if err != nil 
  {
    log.Fatal("Error starting tcp server : ", err)
  }
  defer listener.Close()
  log.Println("Listening on " + CONN_HOST + ":" + CONN_PORT)
  for 
  {
    conn, err := listener.Accept()
    if err != nil 
    {
      log.Fatal("Error accepting: ", err.Error())
    }
    go handleRequest(conn)
  }
}
func handleRequest(conn net.Conn) 
{
  message, err := bufio.NewReader(conn).ReadString('\n')
  if err != nil 
  {
    fmt.Println("Error reading:", err.Error())
  }
  fmt.Print("Message Received from the client: ", string(message))
  conn.Close()
}
```

- 2.Rode o seguinte comando no terminal

```$ go run tcp-server-read-data.go```

### Como isso deve funcionar:
Assim que executarmos o programa, o servidor TCP começará a escutar localmente na porta 8080. A execução de um comando echo a partir da linha de comando da seguinte maneira enviará uma mensagem ao servidor TCP:

```$ echo -n "Hello to TCP server\n" | nc localhost 8080```

- 1.Primeiro, chamamos ```handleRequest``` do método ```main ()``` usando a palavra-chave ```go```, o que significa que estamos invocando uma função em um Goroutine:

```
func main() 
{
  ...
  go handleRequest(conn)
  ...
}
```

- 2.Em seguida, definimos a função ```handleRequest```, que lê uma conexão de entrada no buffer até a primeira ocorrência de \ n e imprime a mensagem no console. Se houver algum erro na leitura da mensagem, a mensagem de erro será impressa junto com o objeto de erro e, finalmente, fechará a conexão, da seguinte forma:

```
func handleRequest(conn net.Conn) 
{
  message, err := bufio.NewReader(conn).ReadString('\n')
  if err != nil 
  {
    fmt.Println("Error reading:", err.Error())
  }
  fmt.Print("Message Received: ", string(message))
  conn.Close()
}
```

## Escrevendo dados para uma conexão TCP
Outro cenário comum, bem importante, em qualquer aplicativo da Web é enviar os dados de volta ao cliente ou responder ao cliente. Go fornece uma maneira conveniente de escrever uma mensagem em uma conexão como bytes.

Como já criamos um servidor TCP que lê dados de conexão de entrada no passo anterior, apenas os atualizamos para gravar a mensagem de volta ao cliente.

Vamos atualizar o método handleRequest no programa para gravar dados no cliente. Execute:

- 1.Crie ```tcp-server-write-data.go``` e copie o seguinte conteúdo:

```
package main
import 
(
  "bufio"
  "fmt"
  "log"
  "net"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
  CONN_TYPE = "tcp"
)
func main() 
{
  listener, err := net.Listen(CONN_TYPE, CONN_HOST+":"+CONN_PORT)
  if err != nil 
  {
    log.Fatal("Error starting tcp server : ", err)
  }
  defer listener.Close()
  log.Println("Listening on " + CONN_HOST + ":" + CONN_PORT)
  for 
  {
    conn, err := listener.Accept()
    if err != nil 
    {
      log.Fatal("Error accepting: ", err.Error())
    }
    go handleRequest(conn)
  }
}
func handleRequest(conn net.Conn) 
{
  message, err := bufio.NewReader(conn).ReadString('\n')
  if err != nil 
  {
    fmt.Println("Error reading: ", err.Error())
  }
  fmt.Print("Message Received:", string(message))
  conn.Write([]byte(message + "\n"))
  conn.Close()
}
```

- 2.No terminal use o comando:

```$ go run tcp-server-write-data.go```

#### Como isso deve funcionar:
Assim que executarmos o programa, o servidor TCP iniciará a escuta local na porta 8080. Execute um comando echo na linha de comando, da seguinte maneira:

```$ echo -n "Hello to TCP server\n" | nc localhost 8080```

Isso nos dará a seguinte resposta do servidor:

```Hello TCP server```

Vejamos as mudanças que introduzimos nesse passo para gravar dados no cliente. Tudo em ```handleRequest``` é exatamente o mesmo que na receita anterior, exceto que introduzimos uma nova linha que grava dados como uma matriz de bytes na conexão, da seguinte maneira:

```
func handleRequest (conn net.Conn)
{
   ...
   conn.Write ([] byte (mensagem + "\ n"))
   ...
}
```

## Implementando o roteamento de solicitações HTTP (HTTP request routing)
Na maioria das vezes, você precisa definir mais de uma rota de URL em um aplicativo da Web, o que envolve o mapeamento do caminho da URL para os manipuladores ou recursos. Nesse passo, aprenderemos como podemos implementá-lo no Go.

Definiremos três rotas, como ```/```, /```login``` e /```logout```, juntamente com seus manipuladores. Execute os seguintes passos:

- 1.Crie http-server-basic-routing.go e copie o seguinte código:

```
package main
import 
(
  "fmt"
  "log"
  "net/http"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
func helloWorld(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Hello World!")
}
func login(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Login Page!")
}
func logout(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Logout Page!")
}
func main() 
{
  http.HandleFunc("/", helloWorld)
  http.HandleFunc("/login", login)
  http.HandleFunc("/logout", logout)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 2.No terminal digite o seguinte comando:

```$ go run http-server-basic-routing.go```


#### Como isso deve funcionar:
Uma vez executado o programa, o servidor HTTP iniciará escutando localmente na porta ```8080``` e acessando ```http://localhost:8080/```, ```http://localhost:8080/login``` e ```http://localhost:8080/logout``` a partir de um navegador ou a linha de comando processará a mensagem definida na definição de manipulador correspondente. Por exemplo, execute ```http://localhost:8080/``` na linha de comando, da seguinte maneira:

```$ curl -X GET -i http://localhost:8080/```

Poderíamos também executar ```http://localhost:8080/login``` a partir da linha de comando da seguinte maneira:

```$ curl -X GET -i http://localhost:8080/login```

**Entendendo o que escrevemos:**

- 1.Começamos com a definição de três manipuladores ou recursos da web, como os seguintes:

```
func helloWorld(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Hello World!")
}
func login(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Login Page!")
}
func logout(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Logout Page!")
}
```

Aqui, o ```handler helloWorld``` escreve ```Hello World!``` em um fluxo de resposta HTTP. De maneira semelhante, os manipuladores de ```login``` e ```logout``` escrevem a ```Login Page```! e ```Logout Page```! em um fluxo de resposta HTTP.

- 2.Em seguida, registramos três caminhos de URL (URL Paths) - ```/```, ```/login``` e ```/logout``` com ```DefaultServeMux``` usando ```http.HandleFunc ()```. Se um padrão de URL de solicitação de entrada corresponder a um dos caminhos registrados, o manipulador correspondente será chamado de passagem ```(http.ResponseWriter, *http.Request)``` como um parâmetro para ele, da seguinte maneira:

```
func main() 
{
  http.HandleFunc("/", helloWorld)
  http.HandleFunc("/login", login)
  http.HandleFunc("/logout", logout)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

##Implementando o roteamento de solicitações HTTP usando o Gorilla Mux
O pacote ```net/http``` do Go oferece muitas funcionalidades para o roteamento de URL das solicitações HTTP. Uma coisa que não faz muito bem é o roteamento de URL dinâmico. Felizmente, podemos conseguir isso com o pacote ```gorilla/mux```, que será abordado nessa parte.

Usaremos ```gorilla/mux``` para definir algumas rotas, como fizemos em nossa receita anterior, junto com seus manipuladores ou recursos. Como já vimos antes, para usar pacotes externos, primeiro temos que instalar o pacote usando o comando ```go get``` ou temos que copiá-lo manualmente para ```$ GOPATH/src``` ou ```$ GOPATH```. Nós faremos o mesmo na receita também. Execute os seguintes passos:

- 1.Instale github.com/gorilla/mux usando o comando go get, da seguinte maneira:

```$ go get github.com/gorilla/mux```

- 2.Crie ```http-server-gorilla-mux-routing.go``` e copie o seguinte código:

```
package main
import 
(
  "net/http"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
var GetRequestHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    w.Write([]byte("Hello World!"))
  }
)
var PostRequestHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    w.Write([]byte("It's a Post Request!"))
  }
)
var PathVariableHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    vars := mux.Vars(r)
    name := vars["name"]
    w.Write([]byte("Hi " + name))
  }
)
func main() 
{
  router := mux.NewRouter()
  router.Handle("/", GetRequestHandler).Methods("GET")
  router.Handle("/post", PostRequestHandler).Methods("POST")
  router.Handle("/hello/{name}", 
  PathVariableHandler).Methods("GET", "PUT")
  http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
}
```

- 3.No terminal rode o seguinte comando:

```$ go run http-server-gorilla-mux-routing.go```

####Como isso deve funcionar:
Uma vez executado o programa, o servidor HTTP iniciará escutando localmente na porta 8080 e acessando ```http://localhost:8080/```, ```http://localhost:8080/post``` e ```http://localhost:8080/hello/foo``` a partir de um navegador ou linha de comando produzirá a mensagem definida na definição de manipulador correspondente. Por exemplo, execute ```http://localhost:8080/``` na linha de comando, da seguinte maneira:

```$ curl -X GET -i http://localhost:8080/```

Poderíamos também executar ```http://localhost:8080/hello/foo``` a partir da linha de comando, da seguinte maneira:

```$ curl -X GET -i http://localhost:8080/hello/foo```

**Entendendo o que escrevemos**

- 1.Primeiro, definimos ```GetRequestHandler``` e ```PostRequestHandler```, que simplesmente escrevem uma mensagem em um fluxo de resposta HTTP, da seguinte forma:

```
var GetRequestHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    w.Write([]byte("Hello World!"))
  }
)
var PostRequestHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    w.Write([]byte("It's a Post Request!"))
  }
)
```

- 2.Em seguida, definimos ```PathVariableHandler```, que extrai variáveis de caminho de solicitação, obtém o valor e grava-o em um fluxo de resposta HTTP, da seguinte maneira:

```
var PathVariableHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    vars := mux.Vars(r)
    name := vars["name"]
    w.Write([]byte("Hi " + name))
  }
)
```

- 3.Em seguida, registramos todos esses manipuladores com o roteador ```gorilla/mux``` e os instanciámos, chamando o manipulador ```NewRouter()``` do roteador mux, da seguinte forma:

```
func main() 
{
  router := mux.NewRouter()
  router.Handle("/", GetRequestHandler).Methods("GET")
  router.Handle("/post", PostCallHandler).Methods("POST")
  router.Handle("/hello/{name}", PathVariableHandler).
  Methods("GET", "PUT")
  http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
}
```

##Registrar solicitações HTTP (Logging HTTP requests)
Registrar solicitações HTTP sempre é útil para solucionar problemas de um aplicativo da Web, por isso é uma boa ideia registrar uma ```solicitação/resposta``` ou melhor citando ```response/request``` com um nível de mensagem e registro adequados. O Go fornece o pacote de ```log```, que pode nos ajudar a implementar o log em um aplicativo. No entanto, nesta receita, usaremos os manipuladores de criação de log do Gorilla para implementá-lo, pois a biblioteca oferece mais recursos, como o registro em log do Apache Combined Log Format e Apache Common Log Format, que ainda não são suportados pelo pacote Go ```log```.

Como já criamos um servidor HTTP e definimos rotas usando o Gorilla Mux em nossa receita anterior, vamos atualizá-lo para incorporar os manipuladores de registro do Gorilla.

Vamos implementar o log usando manipuladores do Gorilla. Execute os seguintes passos:

- 1.Instale os pacotes ```github.com/gorilla/handlere``` e ```github.com/gorilla/mux``` usando o ```go get``` comando da seguinte forma:

```
$ go get github.com/gorilla/handlers 
$ go get github.com/gorilla/mux
```

- 2.Crie ```http-server-request-logging.go``` e copie o seguinte código:

```
package main
import 
(
  "net/http"
  "os"
  "github.com/gorilla/handlers"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
var GetRequestHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    w.Write([]byte("Hello World!"))
  }
)
var PostRequestHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    w.Write([]byte("It's a Post Request!"))
  }
)
var PathVariableHandler = http.HandlerFunc
(
  func(w http.ResponseWriter, r *http.Request) 
  {
    vars := mux.Vars(r)
    name := vars["name"]
    w.Write([]byte("Hi " + name))
  }
)
func main() 
{
  router := mux.NewRouter()
  router.Handle("/", handlers.LoggingHandler(os.Stdout,
  http.HandlerFunc(GetRequestHandler))).Methods("GET")
  logFile, err := os.OpenFile("server.log",
  os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0666)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
  router.Handle("/post", handlers.LoggingHandler(logFile,
  PostRequestHandler)).Methods("POST")
  router.Handle("/hello/{name}",
  handlers.CombinedLoggingHandler(logFile,
  PathVariableHandler)).Methods("GET")
  http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
}
```

- 3.Rode o seguinte comando no terminal:

```$ go run http-server-request-logging.go```

###Como isso deve funcionar:
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

- Execute ```GET``` na linha de comando, da seguinte maneira:

```$ curl -X GET -i http://localhost:8080/```

- Poderíamos também executar a http://localhost:8080/hello/foopartir da linha de comando, da seguinte maneira:

```$ curl -X GET -i http://localhost:8080/ola/foo```

**Entendendo o que fizemos**

- 1.Primeiramente, nós importamos dois pacotes adicionais, um é os, que usamos para abrir um arquivo. O outro é ```github.com/gorilla/handlers```, que usamos para importar manipuladores de log para registrar solicitações HTTP, da seguinte maneira:

```import ("net / http" "os" "github.com/gorilla/handlers" "github.com/gorilla/mux")``` 

- 2.Em seguida, modificamos o método ```main()```. Usando ```router.Handle("/", handlers.LoggingHandler (os.Stdout,
http.HandlerFunc (GetRequestHandler))).Methods("GET")```, envolvemos ```GetRequestHandler``` com um manipulador de criação de log do Gorilla e passamos um fluxo de saída padrão como um gravador para ele, o que significa que estamos simplesmente solicitando que registremos todas as solicitações com o URL caminho / no console no Apache Common Log Format.

- 3.Em seguida, criamos um novo arquivo chamado ```server.log``` no modo somente gravação ou o abrimos, se já existir. Se houver algum erro, registre-o e saia com um código de status 1, da seguinte maneira:

```
logFile, err: = os.OpenFile ("server.log", os.O_WRONLY | os.O_CREATE | os.O_APPEND, 0666) 
se err! = nil 
{ 
  log.Fatal ("erro iniciando servidor http:", err) 
  retornar 
}
```

- 4.Usando ```router.Handle("/ post", handlers.LoggingHandler(logFile, PostRequestHandler)).Métodos("POST")```, envolvemos ```GetRequestHandler``` com um manipulador de logging do Gorilla e passamos o arquivo como um escritor para ele, o que significa que somos simplesmente pedindo para registrar todas as solicitações com o ```caminho/postagem``` (```path/post```) da URL em um arquivo chamado ```/hello/{name}``` no Apache Common Log Format.

- 5.Usando ```router.Handle("/ hello / {name}"```, ```handlers.CombinedLoggingHandler(logFile, PathVariableHandler)).Methods("GET")```, nós empacotamos ```GetRequestHandler``` com um manipulador de logging do Gorilla e passamos o arquivo como um escritor para ele, que significa que estamos simplesmente pedindo para registrar todas as solicitações com o caminho da URL ```/hello/{name}``` em um arquivo denominado ```server.log``` no Apache Combined Log Format.
