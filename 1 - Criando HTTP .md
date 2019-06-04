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

Entenda o que cada linha significa:
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

Entenda o que cada linha significa:
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
