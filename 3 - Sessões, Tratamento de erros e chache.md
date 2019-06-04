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


