# Sessões, tratamento de erros e cache
Às vezes, gostaríamos de persistir informações como dados do usuário em um nível de aplicativo, em vez de persistir em um banco de dados, o que pode ser facilmente alcançado usando sessões e cookies. A diferença entre os dois é que as sessões são armazenadas no lado do servidor, enquanto os cookies são armazenados no lado do cliente. Também podemos precisar armazenar em cache dados estáticos para evitar chamadas desnecessárias para um banco de dados ou um serviço da Web e implementar o tratamento de erros durante o desenvolvimento de um aplicativo da Web. Com o conhecimento dos conceitos abordados neste capítulo, poderemos implementar todas essas funcionalidades de maneira bastante fácil.

Começaremos com a criação de uma sessão HTTP e aprenderemos como podemos gerenciá-lo usando Redis, criando cookies, armazenando em cache respostas HTTP, implementando manipulação de erros e, por fim, finalizando com a implementação de mecanismos de login e logout no Go


## Criando a sua primeira sessão
HTTP é um protocolo sem estado, o que significa que cada vez que um cliente recupera uma página da Web, o cliente abre uma conexão separada para o servidor e o servidor responde a ele sem manter nenhum registro da solicitação do cliente anterior. Portanto, se quisermos implementar um mecanismo no qual o servidor saiba sobre uma solicitação que o cliente enviou para ele, podemos implementá-lo usando uma sessão.

Quando estamos trabalhando com sessões, os clientes precisam apenas enviar um ID e os dados são carregados do servidor para o ID correspondente. Existem três maneiras de implementar isso em um aplicativo da Web:
- Cookies 
- Hidden form fields 
- URL rewrinting
Vamos implementar uma sessão usando cookies HTTP.

- 1.Instale o pacote ```github.com/gorilla/sessions``` usando o comando go get

```$ go get github.com/gorilla/sessions```

- 2.Crie ```http-session.go``` onde criaremos um armazenamento de cookies do Gorilla para salvar e recuperar informações da sessão definindo três manipuladores - ```/login, /home ```e ```/logout``` - onde criaremos um cookie de sessão válido, escrevendo uma resposta para um HTTP fluxo de resposta e invalidando um cookie de sessão, respectivamente.

```
package main
import 
(
  "fmt"
  "log"
  "net/http"
  "github.com/gorilla/sessions"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
var store *sessions.CookieStore
func init() 
{
  store = sessions.NewCookieStore([]byte("secret-key"))
}
func home(w http.ResponseWriter, r *http.Request) 
{
  session, _ := store.Get(r, "session-name")
  var authenticated interface{} = session.Values["authenticated"]
  if authenticated != nil 
  {
    isAuthenticated := session.Values["authenticated"].(bool)
    if !isAuthenticated 
    {
      http.Error(w, "You are unauthorized to view the page",
      http.StatusForbidden)
      return
    }
    fmt.Fprintln(w, "Home Page")
  } 
  else 
  {
    http.Error(w, "You are unauthorized to view the page",
    http.StatusForbidden)
    return
  }
}
func login(w http.ResponseWriter, r *http.Request) 
{
  session, _ := store.Get(r, "session-name")
  session.Values["authenticated"] = true
  session.Save(r, w)
  fmt.Fprintln(w, "You have successfully logged in.")
}
func logout(w http.ResponseWriter, r *http.Request) 
{
  session, _ := store.Get(r, "session-name")
  session.Values["authenticated"] = false
  session.Save(r, w)
  fmt.Fprintln(w, "You have successfully logged out.")
}
func main() 
{
  http.HandleFunc("/home", home)
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

- 3.Rode no terminal

```$ go run http-session.go```

### Como isso deve funcionar
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

- Em seguida, executaremos alguns comandos para ver como a sessão funciona.

- Primeiro, vamos acessar ```/home``` executando o seguinte comando:

```$ curl -X GET http://localhost:8080/home```

- Isso ocorre porque primeiro precisamos fazer login em um aplicativo, o que criará um ID de sessão que o servidor validará antes de fornecer acesso a qualquer página da web. Então, vamos entrar no aplicativo:

```$ curl -X GET -i http://localhost:8080/login```

- Em seguida, usaremos esse cookie fornecido para acessar ```/home```, da seguinte maneira:

```$ curl --cookie "session-name=MTUyMzEwMTI3NXxEdi1CQkFFQ180SUFBUkFCRUFBQUpmLUNBQUVHYzNSeWFXNW5EQThBRFdGMWRHaGxiblJwWTJGMFpXUUVZbTl2YkFJQ0FBRT18ou7Zxn3qSbqHHiajubn23Eiv8a348AhPl8RN3uTRM4M=;" http://localhost:8080/home```

**Entendendo as partes**
- Usando ```var store * sessions.CookieStore```, declaramos uma loja particular de cookies para armazenar sessões usando cookies seguros.

- Usando ```func init () {store = sessions.NewCookieStore ([] byte ("chave secreta"))}```, definimos uma função ```init()``` que é executada antes de ```main()``` para criar um novo armazenamento de cookie e atribuí-lo ao armazenamento.

> O ```init()``` é sempre chamado, independentemente de haver uma função principal ou não, portanto, se você importar um pacote que tenha uma função init, ela será executada.

- Em seguida, definimos um manipulador ```home``` onde obtemos uma sessão do armazenamento de cookies para o nome fornecido depois de adicioná-lo ao registro usando ```store.Get``` e buscar o valor da chave autenticada do cache. Se for verdade, então escrevemos Home Page para um fluxo de resposta HTTP; caso contrário, nós escrevemos um Você não está autorizado a ver a página. mensagem junto com um código HTTP 403.

- Em seguida, definimos um manipulador de login em que novamente obtemos uma sessão, definimos a chave autenticada com um valor true, salvamos e finalmente gravamos Você efetuou login com êxito em um fluxo de resposta HTTP.

-Em seguida, definimos um manipulador de logout no qual obtemos uma sessão, definimos uma chave autenticada com o valor false, salvamos e, finalmente, escrevemos Você efetuou logout com sucesso. para um fluxo de resposta HTTP.
Finalmente, definimos ```main()``` onde mapeamos todos os manipuladores, home, login e logout, para ```/home, /login e /logout```, respectivamente, e iniciamos o servidor HTTP em ```localhost: 8080```.


## Gerenciando sua sessão HTTP usando o Redis
Ao trabalhar com os aplicativos distribuídos, provavelmente temos que implementar o balanceamento de carga sem estado para os usuários front-end. Isso é para que possamos manter as informações da sessão em um banco de dados ou em um sistema de arquivos para que possamos identificar o usuário e recuperar suas informações se um servidor for desligado ou reiniciado.

Nós resolveremos esse problema usando o Redis como o armazenamento persistente para salvar uma sessão.

Como já criamos uma variável de sessão em nossa receita anterior usando o armazenamento de cookies do Gorilla, vamos apenas estender esta receita para salvar as informações da sessão no Redis em vez de mantê-las no servidor.

Existem várias implementações do armazenamento de sessão do Gorilla, que você pode encontrar em ```https://github.com/gorilla/sessions#store-implementations```. Como estamos usando o Redis como nosso back-end, estaremos usando ```https://github.com/boj/redistore```, que depende da biblioteca Redigo Redis para armazenar uma sessão.

Você tem o Redis e o Redis Browser instalados e sendo executados localmente nas portas ```6379``` e ```4567```, respectivamente.

- 1.Instale ```gopkg.in/boj/redistore.v1``` e ```github.com/gorilla/sessions``` usando o comando ```go get```

```
$ go get gopkg.in/boj/redistore.v1
$ go get github.com/gorilla/sessions
```

- 2.Crie ```http-session-redis.go```, onde criaremos um ```RedisStore``` para armazenar e recuperar variáveis de sessão:

```
package main
import 
(
  "fmt"
  "log"
  "net/http"
  "github.com/gorilla/sessions"
  redisStore "gopkg.in/boj/redistore.v1"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
var store *redisStore.RediStore
var err error
func init() 
{
  store, err = redisStore.NewRediStore(10, "tcp", ":6379", "",
  []byte("secret-key"))
  if err != nil 
  {
    log.Fatal("error getting redis store : ", err)
  }
}
func home(w http.ResponseWriter, r *http.Request) 
{
  session, _ := store.Get(r, "session-name")
  var authenticated interface{} = session.Values["authenticated"]
  if authenticated != nil 
  {
    isAuthenticated := session.Values["authenticated"].(bool)
    if !isAuthenticated 
    {
      http.Error(w, "You are unauthorized to view the page",
      http.StatusForbidden)
      return
    }
    fmt.Fprintln(w, "Home Page")
  } 
  else 
  {
    http.Error(w, "You are unauthorized to view the page",
    http.StatusForbidden)
    return
  }
}
func login(w http.ResponseWriter, r *http.Request) 
{
  session, _ := store.Get(r, "session-name")
  session.Values["authenticated"] = true
  if err = sessions.Save(r, w); err != nil 
  {
    log.Fatalf("Error saving session: %v", err)
  }
  fmt.Fprintln(w, "You have successfully logged in.")
}
func logout(w http.ResponseWriter, r *http.Request) 
{
  session, _ := store.Get(r, "session-name")
  session.Values["authenticated"] = false
  session.Save(r, w)
  fmt.Fprintln(w, "You have successfully logged out.")
}
func main() 
{
  http.HandleFunc("/home", home)
  http.HandleFunc("/login", login)
  http.HandleFunc("/logout", logout)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  defer store.Close()
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run http-session-redis.go```


### Como isso deve funcionar
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

- Em seguida, executaremos alguns comandos para ver como a sessão funciona.

- Primeiro, vamos acessar ```/home``` executando o seguinte comando:

```$ curl -X GET http://localhost:8080/home```

- Isso resultará em uma mensagem de acesso não autorizada do servidor

- Isso ocorre porque primeiro precisamos fazer login em um aplicativo, que criará um ID de sessão que o servidor validará antes de fornecer acesso a qualquer página da web. Então, vamos entrar no aplicativo:

```$ curl -X GET -i http://localhost:8080/login```

- O comando anterior nos dará o Cookie, que deve ser configurado como um cabeçalho de solicitação para acessar qualquer página da web 

- Uma vez que o comando anterior é executado, um Cookie será criado e salvo no Redis, que você pode ver executando o comando do redis-cli ou no Navegador Redis

- Usaremos o cookie fornecido para acessar / home, da seguinte maneira

```$ curl --cookie "session-name=MTUyMzEwNDUyM3xOd3dBTkV4T1JrdzNURFkyUkVWWlQxWklUekpKVUVOWE1saFRUMHBHVTB4T1RGVXlSRU5RVkZWWk5VeFNWVmRPVVZSQk4wTk1RMUU9fAlGgLGU-OHxoP78xzEHMoiuY0Q4rrbsXfajSS6HiJAm;" http://localhost:8080/home```

- Isso resultará na home page como uma resposta do servidor

**Entendendo as partes**
- Usando ```var store *redisStore.RediStore```, declaramos uma ```RediStore``` privada para armazenar sessões no Redis

-Em seguida, atualizamos a função ```init()``` para criar ```NewRediStore``` com um tamanho e um número máximo de conexões ociosas como 10 e atribuímos a ela ao armazenamento. Se houver um erro ao criar um armazenamento, registramos o erro e saímos com um código de status igual a 1

-Finalmente, atualizamos ```main()``` para introduzir a instrução defer ```store.Close()```, que fecha o repositório Redis assim que retornamos da função


## Seu primeiro Cookie
Os cookies desempenham um papel importante ao armazenar informações no lado do cliente e podemos usar seus valores para identificar um usuário. Basicamente, os cookies foram inventados para resolver o problema de lembrar informações sobre o usuário ou autenticação de login persistente, que se refere a sites que são capazes de lembrar a identidade de um principal entre as sessões.
Cookies são arquivos de texto simples que os navegadores da Web criam quando você visita sites na Internet. Seu dispositivo armazena os arquivos de texto localmente, permitindo que o navegador acesse o cookie e passe os dados de volta ao site original e sejam salvos em pares de nome-valor.

- 1.Instale o pacote ```github.com/gorilla/securecookie``` usando o comando ```go get```

```$ go get github.com/gorilla/securecookie```

- 2.Crie http-cookie.go, onde criaremos um cookie seguro do Gorilla para armazenar e recuperar cookies assim:

```
package main
import 
(
  "fmt"
  "log"
  "net/http"
  "github.com/gorilla/securecookie"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
var cookieHandler *securecookie.SecureCookie
func init() 
{
  cookieHandler = securecookie.New(securecookie.
  GenerateRandomKey(64),
  securecookie.GenerateRandomKey(32))
}
func createCookie(w http.ResponseWriter, r *http.Request) 
{
  value := map[string]string
  {
    "username": "Foo",
  }
  base64Encoded, err := cookieHandler.Encode("key", value)
  if err == nil 
  {
    cookie := &http.Cookie
    {
      Name: "first-cookie",
      Value: base64Encoded,
      Path: "/",
    }
    http.SetCookie(w, cookie)
  }
  w.Write([]byte(fmt.Sprintf("Cookie created.")))
}
func readCookie(w http.ResponseWriter, r *http.Request) 
{
  log.Printf("Reading Cookie..")
  cookie, err := r.Cookie("first-cookie")
  if cookie != nil && err == nil 
  {
    value := make(map[string]string)
    if err = cookieHandler.Decode("key", cookie.Value, &value); 
    err == nil 
    {
      w.Write([]byte(fmt.Sprintf("Hello %v \n", 
      value["username"])))
    }
  } 
  else 
  {
    log.Printf("Cookie not found..")
    w.Write([]byte(fmt.Sprint("Hello")))
  }
}

func main() 
{
  http.HandleFunc("/create", createCookie)
  http.HandleFunc("/read", readCookie)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run http-cookie.go```


### Como isso deve funcionar
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

- A navegação ```http://localhost:8080/read``` exibirá Hello no navegador

- Em seguida, acessaremos ```http://localhost:8080/create```, que criará um cookie com o nome ```first-cookie``` e exibirá a mensagem **Cookie created** no navegador

- O acesso a ```http://localhost:8080/read``` usará o primeiro cookie para exibir Hello, seguido pelo valor do primeiro cookie

**Entendendo as partes**
- Usando ```import ("fmt" "log" "net/http" "github.com/gorilla/securecookie ")```, introduzimos um pacote adicional - ```github.com/gorilla/securecookie```, que usaremos para codificar e decodificar valores de cookies autenticados e criptografados.

- Usando ```var cookieHandler * securecookie.SecureCookie```, declaramos um cookie seguro privado.

- Em seguida, atualizamos a função ```init()``` para criar o SecureCookie passando uma chave hash de 64 bytes, que é usada para autenticar valores usando o HMAC e uma chave de bloco de 32 bytes, que é usada para criptografar valores.

- Em seguida, definimos um manipulador ```createCookie``` onde criamos um cookie codificado em ```Base64``` com a chave como ```username``` e o valor como ```Foo``` usando ```Encode``` de ```gorilla/securecookie```. Em seguida, adicionamos um cabeçalho Set-Cookie aos cabeçalhos ResponseWriter fornecidos e escrevemos um Cookie criado. mensagem para uma resposta HTTP.

- Em seguida, definimos um manipulador ```readCookie```, onde recuperamos um cookie da solicitação, que é o primeiro cookie em nosso código, obtemos um valor para ele e o gravamos em uma resposta HTTP.

- Finalmente, definimos ```main()``` onde mapeamos todos os manipuladores - ```createCookie``` e ```readCookie``` - para ```/create``` e ```/read```, respectivamente, e iniciamos o servidor HTTP em ```localhost:8080```.


## Implementação de cache
O armazenamento em cache de dados em um aplicativo da Web às vezes é necessário para evitar a solicitação repetida de dados estáticos de um banco de dados ou serviço externo. O Go não fornece nenhum pacote interno para armazenar respostas em cache, mas o suporta por meio de pacotes externos.
Existem vários pacotes, como ```https://github.com/coocood/freecache``` e ```https://github.com/patrickmn/go-cache```, que podem ajudar na implementação do cache e, nesta parte, usaremos o ```https://github.com/patrickmn/go-cache``` para implementá-lo.

- 1.Instale o pacote ```github.com/patrickmn/go-cache``` usando o comando ```go get```

```$ go get github.com/patrickmn/go-cache```

- 2.Crie ```http-caching.go```, onde criaremos um cache e o preencheremos com dados de inicialização do servidor

```
package main
import 
(
  "fmt"
  "log"
  "net/http"
  "time"
  "github.com/patrickmn/go-cache"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
var newCache *cache.Cache
func init() 
{
  newCache = cache.New(5*time.Minute, 10*time.Minute)
  newCache.Set("foo", "bar", cache.DefaultExpiration)
}
func getFromCache(w http.ResponseWriter, r *http.Request) 
{
  foo, found := newCache.Get("foo")
  if found 
  {
    log.Print("Key Found in Cache with value as :: ", 
    foo.(string))
    fmt.Fprintf(w, "Hello "+foo.(string))
  } 
  else 
  {
    log.Print("Key Not Found in Cache :: ", "foo")
    fmt.Fprintf(w, "Key Not Found in Cache")
  }
}
func main() 
{
  http.HandleFunc("/", getFromCache)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run http-caching.go```


### Como isso deve funcionar
-  Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta 8080

- Na inicialização, a chave com o nome foo com um valor como barra será adicionada ao cache

- Navegar ```http://localhost:8080/``` lerá um valor de chave do cache e o anexará a Hello

- Especificamos o tempo de expiração dos dados do cache em nosso programa como cinco minutos, o que significa que a chave que criamos no cache na inicialização do servidor não estará lá depois de cinco minutos. Portanto, acessar a mesma URL novamente após cinco minutos retornará a chave não encontrada no cache do servidor

**Entendendo as partes**
- Usando ```var newCache * cache.Cache```, declaramos um cache privado

- Em seguida, atualizamos a função ```init()``` onde criamos um cache com cinco minutos de tempo de expiração e 10 minutos de intervalo de limpeza, e adicionamos um item ao cache com uma chave como ```foo``` com seu valor como bar e seu valor de expiração como 0 , o que significa que queremos usar o tempo de expiração padrão do cache
> Se a duração da expiração for menor que um (ou ```NoExpiration```), os itens no cache nunca expiram (por padrão) e devem ser excluídos manualmente. Se o intervalo de limpeza for menor que um, os itens expirados não serão excluídos do cache antes de chamar ```c.DeleteExpired()```

- Definimos o manipulador ```getFromCache``` onde recuperamos o valor de uma chave do cache. Se encontrado, nós o escrevemos em uma resposta HTTP; caso contrário, gravamos a mensagem ```Chave não encontrada``` | ```Key Not Found in Cache``` no cache em uma resposta HTTP


## Implementando o tratamento de erros HTTP no Go (HTTP error handling in Go)
A implementação do tratamento de erros em qualquer aplicativo da Web é um dos principais aspectos, pois ajuda na solução de problemas e na correção de erros mais rapidamente. Manipulação de erros significa sempre que um erro ocorre em um aplicativo, ele deve ser registrado em algum lugar, em um arquivo ou em um banco de dados com a mensagem de erro adequada, junto com o rastreamento de pilha.
Em Go, pode ser implementado de várias maneiras. Uma maneira é escrever manipuladores personalizados.

- 1.Instale o pacote ```github.com/gorilla/mux``` usando o comando ```go get```

```$ go get github.com/gorilla/mux```

- 2.Crie ```http-error-handling.go```, onde criaremos um manipulador personalizado que atua como um ```wrapper``` para lidar com todas as solicitações HTTP

```
package main
import 
(
  "errors"
  "fmt"
  "log"
  "net/http"
  "strings"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type NameNotFoundError struct 
{
  Code int
  Err error
}
func (nameNotFoundError NameNotFoundError) Error() string 
{
  return nameNotFoundError.Err.Error()
}
type WrapperHandler func(http.ResponseWriter, *http.Request) 
error
func (wrapperHandler WrapperHandler) ServeHTTP(w http.
ResponseWriter, r *http.Request) 
{
  err := wrapperHandler(w, r)
  if err != nil 
  {
    switch e := err.(type) 
    {
      case NameNotFoundError:
      log.Printf("HTTP %s - %d", e.Err, e.Code)
      http.Error(w, e.Err.Error(), e.Code)
      default:
      http.Error(w, http.StatusText(http.
      StatusInternalServerError),
      http.StatusInternalServerError)
    }
  }
}
func getName(w http.ResponseWriter, r *http.Request) error 
{
  vars := mux.Vars(r)
  name := vars["name"]
  if strings.EqualFold(name, "foo") 
  {
    fmt.Fprintf(w, "Hello "+name)
    return nil
  } 
  else 
  {
    return NameNotFoundError{500, errors.New("Name Not Found")}
  }
}
func main() 
{
  router := mux.NewRouter()
  router.Handle("/employee/get/{name}",
  WrapperHandler(getName)).Methods("GET")
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.Rode no terminal

```$ go run http-error-handling.go```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta 8080

- Em seguida, navegando ```http://localhost:8080/employee/get/foo``` nos dará o Hello, seguido pelo nome do funcionário com o código de status como 200, como uma resposta no navegador

- Acessando ```http://localhost:8080/employee/get/bar``` nos retornará um erro HTTP com a mensagem ```Name Not Found``` e um código de erro de 500

**Entendendo as partes**
- Definimos uma estrutura ```NameNotFoundError``` com dois campos - Código do tipo int e erro do tipo error, que representa um erro com um código de status HTTP associado

```
type NameNotFoundError struct 
{
  Code int
  Err error
}
```
- Permitimos que ```NameNotFoundError``` satisfizesse a interface de erro

```
func (nameNotFoundError NameNotFoundError) Error() string 
{
  return nameNotFoundError.Err.Error()
}
```

- Em seguida, definimos um tipo ```WrapperHandler``` definido pelo usuário, que é uma função Go que aceita qualquer manipulador que aceita ```func (http.ResponseWriter, * http.Request)``` como parâmetros de entrada e retorna um erro

- Em seguida, definimos um manipulador ```ServeHTTP```, que chama um manipulador que passamos para ```WrapperHandler passing (http.ResponseWriter, * http.Request)``` como parâmetros para ele e verifica se há algum erro retornado pelo manipulador. Se houver, então os manipulará apropriadamente usando o caso de troca

```
if err != nil 
{
  switch e := err.(type) 
  {
    case NameNotFoundError:
    log.Printf("HTTP %s - %d", e.Err, e.Code)
    http.Error(w, e.Err.Error(), e.Code)
    default:
    http.Error(w, http.StatusText(http.
    StatusInternalServerError),
    http.StatusInternalServerError)
  }
}
```

- Definimos um manipulador ```getName```, que extrai variáveis de caminho de solicitação, obtém o valor da variável ```name``` e verifica se o nome corresponde a ```foo```. Em caso afirmativo, ele escreve Hello, seguido do nome, para uma resposta HTTP; caso contrário, ele retornará uma estrutura ```NameNotFoundError``` com um valor de campo Código de 500 e um valor de campo err de um erro com o texto Nome não encontrado ```(Name not found)```

Finalmente, definimos ```main()```, onde registramos ```WrapperHandler``` como um manipulador a ser chamado para o padrão de URL ```como /get/{name}```

## Implementando login e logout no aplicativo da web
Sempre que queremos que um aplicativo seja acessado por usuários registrados, precisamos implementar um mecanismo que solicite as credenciais do usuário antes de permitir que eles visualizem quaisquer páginas da Web

Como já criamos um formulário HTML em uma de nossas receitas anteriores, vamos apenas atualizá-lo para implementar mecanismos de login e logout usando o pacote ```gorilla/securecookie```

- 1.Instale ```github.com/gorilla/mux``` e ```github.com/gorilla/securecookie``` usando o comando ```go get```

```
$ go get github.com/gorilla/mux
$ go get github.com/gorilla/securecookie
```

- 2.Crie ```home.html``` dentro do diretório de templates

```$ mkdir templates && cd templates && touch home.html```

- 3.Copie o conteúdo para ```home.html```

```
<html>
  <head>
    <title></title>
  </head>
  <body>
    <h1>Welcome {{.userName}}!</h1>
    <form method="post" action="/logout">
      <button type="submit">Logout</button>
    </form>
  </body>
</html>
```

> No modelo anterior, definimos um marcador, ```{{.userName}}```, cujos valores serão substituídos pelo mecanismo de modelo no tempo de execução e um botão Logout. Ao clicar no botão Logout, o cliente fará uma chamada POST para uma ação de formulário, que é ```/logout```

- 4.Crie ```html-form-login-logout.go```, onde analisaremos o formulário de login, leremos o campo de nome de usuário e definiremos um cookie de sessão quando um usuário clicar no botão Login. Também limpamos a sessão quando um usuário clica no botão Logout

```
package main
import 
(
  "html/template"
  "log"
  "net/http"
  "github.com/gorilla/mux"
  "github.com/gorilla/securecookie"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
var cookieHandler = securecookie.New
(
  securecookie.GenerateRandomKey(64),
  securecookie.GenerateRandomKey(32)
)
func getUserName(request *http.Request) (userName string) 
{
  cookie, err := request.Cookie("session")
  if err == nil 
  {
    cookieValue := make(map[string]string)
    err = cookieHandler.Decode("session", cookie.Value,
    &cookieValue)
    if err == nil 
    {
      userName = cookieValue["username"]
    }
  }
  return userName
}
func setSession(userName string, response http.ResponseWriter) 
{
  value := map[string]string
  {
    "username": userName,
  }
  encoded, err := cookieHandler.Encode("session", value)
  if err == nil 
  {
    cookie := &http.Cookie
    {
      Name: "session",
      Value: encoded,
      Path: "/",
    }
    http.SetCookie(response, cookie)
  }
}
func clearSession(response http.ResponseWriter) 
{
  cookie := &http.Cookie
  {
    Name: "session",
    Value: "",
    Path: "/",
    MaxAge: -1,
  }
  http.SetCookie(response, cookie)
}
func login(response http.ResponseWriter, request *http.Request) 
{
  username := request.FormValue("username")
  password := request.FormValue("password")
  target := "/"
  if username != "" && password != "" 
  {
    setSession(username, response)
    target = "/home"
  }
  http.Redirect(response, request, target, 302)
}
func logout(response http.ResponseWriter, request *http.Request) 
{
  clearSession(response)
  http.Redirect(response, request, "/", 302)
}
func loginPage(w http.ResponseWriter, r *http.Request) 
{
  parsedTemplate, _ := template.ParseFiles("templates/
  login-form.html")
  parsedTemplate.Execute(w, nil)
}
func homePage(response http.ResponseWriter, request *http.Request) 
{
  userName := getUserName(request)
  if userName != "" 
  {
    data := map[string]interface{}
    {
      "userName": userName,
    }
    parsedTemplate, _ := template.ParseFiles("templates/home.html")
    parsedTemplate.Execute(response, data)
  } 
  else 
  {
    http.Redirect(response, request, "/", 302)
  }
}
func main() 
{
  var router = mux.NewRouter()
  router.HandleFunc("/", loginPage)
  router.HandleFunc("/home", homePage)
  router.HandleFunc("/login", login).Methods("POST")
  router.HandleFunc("/logout", logout).Methods("POST")
  http.Handle("/", router)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 5.Rode no terminal

```$ go run html-form-login-logout.go```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

- Em seguida, navegando ```http://localhost:8080``` nos mostrará o formulário de login

- Submetendo o formulário depois de digitar o nome de usuário Foo e uma senha aleatória irá renderizar o **Welcome Foo!** mensagem no navegador e crie um cookie com a sessão ```name```, que gerencia o estado de ```login/logout```

- Agora, todas as solicitações subsequentes para ```http://localhost:8080/home``` exibirão o Welcome Foo! mensagem no navegador até que o cookie com o nome da sessão exista

- Em seguida, acessar ```http://localhost:8080/home``` depois de limpar o cookie nos redirecionará para ```http://localhost:8080/``` e nos mostrará o formulário de login

**Entendendo as partes**
- Usando ```var cookieHandler = securecookie.New (securecookie.GenerateRandomKey (64), securecookie.GenerateRandomKey (32))```, estamos criando um cookie seguro, passando uma chave de hash como o primeiro argumento e uma chave de bloco como o segundo argumento. A chave hash é usada para autenticar valores usando o HMAC e a chave de bloqueio é usada para criptografar valores

- Definimos um manipulador ```getUserName```, onde obtemos um cookie da solicitação HTTP, inicializamos um mapa ```cookieValue``` de chaves de string para valores de string, decodificamos um cookie e obtemos um valor para o nome de usuário e retorno

- Definimos um manipulador ```setSession```, onde criamos e inicializamos um mapa com a chave e valor como username, serializamos, assinamos com um código de autenticação de mensagem, codificamos usando um manipulador ```cookieHandler.Encode```, criamos um novo cookie HTTP e gravá-lo em um fluxo de resposta HTTP

- Definimos ```clearSession```, que basicamente define o valor do cookie como vazio e o grava em um fluxo de resposta HTTP

- Um manipulador de ```login```, onde obtemos um nome de usuário e senha de um formulário HTTP, verificamos se ambos não estão vazios, depois chamamos um manipulador ```setSession``` e redirecionamos para ```/home```, caso contrário, redirecionamos para o URL raiz /

- ```Logout``` handler, onde limpamos os valores da sessão chamando o manipulador ```clearSession``` e redirecionamos para a URL raiz

- ```loginPage```, onde analisamos ```login-form.html```, retorna um novo modelo com o nome e seu conteúdo, chama o manipulador ```Execute``` em um modelo analisado, que gera saída HTML e grava-o em um fluxo de resposta HTTP

- ```homePage``` handler, que obtém o nome de usuário da solicitação HTTP chamando o manipulador ```getUserName```. Em seguida, verificamos se não está vazio ou se há um valor de cookie presente. Se o nome de usuário não estiver em branco, analisamos ```home.html```, injetamos o nome de usuário como um mapa de dados, geramos a saída HTML e gravamos em um fluxo de resposta HTTP; caso contrário, redirecionamos para o URL raiz /

- método ```main()```, onde iniciamos a execução do programa. Como esse método faz muitas coisas:

    - ```var router = mux.NewRouter():```criamos uma nova instância do roteador

    - ```router.HandleFunc("/", loginPage):```estamos registrando o manipulador ```loginPageHandler``` com o padrão / URL usando o HandleFunc do pacote ```gorilla / mux```, o que significa que o manipulador ```loginPage``` é executado passando ```(http.ResponseWriter, * http. Pedido)``` como parâmetros para ele sempre que acessar o URL HTTP com o padrão /

    - ```router.HandleFunc("/home", homePage):```estamos registrando o manipulador ```homePageHandler``` com o padrão de URL / home usando o ```HandleFunc``` do pacote ```gorilla / mux```, o que significa que o manipulador ```homePage``` é executado passando (http.ResponseWriter, * http.Request) como parâmetros para ele sempre que acessar o URL HTTP com o padrão / home

    - ```router.HandleFunc("/login", login).Methods("POST"):```estamos registrando o manipulador loginHandler com o padrão de URL / login usando o HandleFunc do pacote gorilla / mux, o que significa que o manipulador de login é executado passando (http.ResponseWriter, * http.Request) como parâmetros para ele sempre que acessar o URL HTTP com o padrão / login

    - ```router.HandleFunc("/logout", logout).Methods("POST"):```estamos registrando o manipulador logoutHandler com o padrão de URL / logout usando o HandleFunc do pacote gorilla / mux, o que significa que o manipulador de logout é executado passando (http.ResponseWriter, * http.Request) como parâmetros para ele sempre que acessar o URL HTTP com o padrão / logout

    - ```http.Handle("/", router):```estamos registrando o roteador para o padrão / URL usando HandleFunc do pacote net / http, o que significa que todas as solicitações com o padrão / URL são manipuladas pelo manipulador do roteador. 

    - ```err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil):```estamos chamando http.ListenAndServe para atender a solicitações HTTP que tratam cada conexão de entrada em um Goroutine separado. O ListenAndServe aceita dois parâmetros - endereço do servidor e manipulador, em que o endereço do servidor é localhost: 8080 e o manipulador é nulo, o que significa que estamos solicitando ao servidor que use o DefaultServeMux como manipulador

    - ```if err != nil { log.Fatal("error starting http server : ", err) return}:```verificamos se há algum problema com o início do servidor. Se houver, registre o erro e saia com um código de status 1
