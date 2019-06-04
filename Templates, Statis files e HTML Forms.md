# Templates, Static Filles e HTML Forms
Muitas vezes, gostaríamos de criar formulários HTML para obter as informações de um cliente em um formato especificado, fazer upload de arquivos ou pastas para o servidor e gerar modelos HTML genéricos, em vez de repetir o mesmo texto estático. Com o conhecimento dos conceitos abordados neste capítulo, seremos capazes de implementar todas essas funcionalidades eficientemente no Go.Neste capítulo, começaremos criando um template básico e depois passaremos a servir arquivos estáticos, como .js, .css e imagens de um sistema de arquivos e, eventualmente, criar, ler e validar formulários HTML e carregar um arquivo no servidor.

## Criando o primeiro template
Os templates nos permitem definir espaços reservados para conteúdo dinâmico que podem ser substituídos pelos valores em tempo de execução por um mecanismo de modelo/template. Eles podem então ser transformados em um arquivo HTML e enviados para o cliente. Criar modelos no Go é bastante fácil usando o pacote ```html/template``` de Go.

Vamos criar um primeiro template.html com alguns espaços reservados cujo valor será injetado pelo mecanismo de modelo no tempo de execução. Execute os seguintes passos:

- 1.Crie ```first-template.html``` dentro do diretório templates (crie esse diretório também), executando o comando:

```$ mkdir templates && cd templates && touch first-template.html```

- 2.Copie o código no ```first-template.html```:

```
<html>
  <head>
    <meta charset="utf-8">
    <title>First Template</title>
    <link rel="stylesheet" href="/static/stylesheets/main.css">
  </head>
  <body>
    <h1>Hello {{.Name}}!</h1>
    Your Id is {{.Id}}
  </body>
</html>
```

O modelo anterior tem dois espaços reservados, ```{{.Name}}``` e ```{{.Id}}```, cujos valores serão substituídos ou injetados pelo mecanismo de modelo no tempo de execução.


- 3.Crie ```first-template.go```, onde preencheremos os valores dos espaços reservados, geraremos um HTML como saída e o escreveremos no cliente, da seguinte maneira:

```
import 
(
  "fmt"
  "html/template"
  "log"
  "net/http"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type Person struct 
{
  Id   string
  Name string
}
func renderTemplate(w http.ResponseWriter, r *http.Request) 
{
  person := Person{Id: "1", Name: "Foo"}
  parsedTemplate, _ := template.ParseFiles("templates/
  first-template.html")
  err := parsedTemplate.Execute(w, person)
  if err != nil 
  {
    log.Printf("Error occurred while executing the template
    or writing its output : ", err)
    return
  }
}
func main() 
{
  http.HandleFunc("/", renderTemplate)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 4.Rode o programa conforme a abaixo:

```$ go run first-template.go```


### Como isso funciona

Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

Navegando ```http://localhost:8080``` nos mostrará o ```Hello Foo!``` que está no modelo

Execute ```curl -X GET http://localhost:8080``` na linha de comando:

```$ curl -X GET http://localhost:8080```

Isso resultará na seguinte resposta do servidor 

**Entendo as partes**

- ```type person struct {Id string Name string}```: Aqui definimos uma pessoa struct type que possui os campos ```Id``` e ```Name```.

> O nome do campo deve começar com uma letra maiúscula na definição de tipo; caso contrário, isso resultará em erros e não será substituído no modelo.
Em seguida, definimos um manipulador "renderTemplate()", que faz muitas coisas.

- ```person := Person {Id: "1", Name: "Foo"}```: Aqui estamos inicializando um tipo de estrutura de pessoa com Id como 1 e Name como Foo.

- ```parsedTemplate, _: = template.ParseFiles("templates / first-template.html")```: Aqui estamos chamando ```ParseFiles``` do pacote ```html/template```, que cria um novo template e analisa o nome do arquivo que passamos como uma entrada, que é ```first-template.html```, em um diretório de modelos. O modelo resultante terá o nome e o conteúdo do arquivo de entrada.

- ```err: = parsedTemplate.Execute(w, person)```: Aqui estamos chamando um manipulador ```Execute``` em um modelo analisado, que injeta dados de pessoas no modelo, gera uma saída HTML e os grava em um fluxo de resposta HTTP.

- ```if err != nil {log.Printf("Error occurred while executing the template or writing its output : ", err) return }```: Aqui nós verificamos se há algum problema ao executar o template ou escrever sua saída no fluxo de resposta. Se houver, registramos o erro e saímos com um código de status 1.


## Servindo arquivos estáticos através de HTTP (Serving static files over HTTP)
Ao projetar aplicativos da web, é sempre uma prática recomendada servir recursos estáticos, como ".js", ".css" e "images", do sistema de arquivos ou de qualquer **rede de entrega de conteúdo (CDN - content delivery network)**, como Akamai ou Amazon CloudFront, em vez de servi-los o servidor da web. Isso ocorre porque todos esses tipos de arquivos são estáticos e não precisam ser processados. Então, por que devemos colocar carga extra no servidor? Além disso, ajuda a aumentar o desempenho do aplicativo, pois todas as solicitações para os arquivos estáticos serão fornecidas por fontes externas e, portanto, reduzirão a carga no servidor.

O ```pacote net/http``` do Go é suficiente para servir recursos estáticos do sistema de arquivos através do FileServer, que será abordado nessa parte.

Como já criamos um modelo no passo anterior, vamos apenas estendê-lo para servir um arquivo .css estático do diretório ```static/css```.

Vamos criar um servidor de arquivos que servirá recursos estáticos do sistema de arquivos. Execute os seguintes passos:

- 1.Crie ```main.css``` dentro de um diretório ```static/css```:


```$ mkdir static && cd static && mkdir css && cd css && touch main.css```

- 2.Copie o conteudo para ```main.css```

```body {color: #00008B}```

- 3.Crie ```server-static-files.go```, onde criaremos ```FileServer```, que servirá recursos do diretório ```static/css``` presente no sistema de arquivos para todos os padrões de URL com ```/static```, da seguinte maneira:

```
package main
import 
(
  "fmt"
  "html/template"
  "log"
  "net/http"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type Person struct 
{
  Name string
  Age string
}
func renderTemplate(w http.ResponseWriter, r *http.Request) 
{
  person := Person{Id: "1", Name: "Foo"}
  parsedTemplate, _ := template.ParseFiles("templates/
  first-template.html")
  err := parsedTemplate.Execute(w, person)
  if err != nil 
  {
    log.Printf("Error occurred while executing the template 
    or writing its output : ", err)
    return
  }
}
func main() 
{
  fileServer := http.FileServer(http.Dir("static"))
  http.Handle("/static/", http.StripPrefix("/static/", fileServer))
  http.HandleFunc("/", renderTemplate)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 4.Atualize ```first-template.html``` (o que você criou) para incluir ```main.css``` do diretório ```static/css```:

```
<html>
  <head>
    <meta charset="utf-8">
    <title>First Template</title>
    <link rel="stylesheet" href="/static/css/main.css">
  </head>
  <body>
    <h1>Hello {{.Name}}!</h1>
    Your Id is {{.Id}}
  </body>
</html>
```

- 5.Rode o programa no terminal

```$ go run serve-static-files.go```


### Como isso deve funcionar
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```. Navegue ```http://localhost:8080``` nos mostrará a mesma saída que vimos anteriormente, mas desta vez a cor do texto mudou do padrão **preto** para **azul**.

Se olharmos para a guia Rede do **Chrome DevTools (Network Chrome DevTool F12)**, podemos ver ```main.css```, que foi carregado do diretório ```static/css``` presente no sistema de arquivos.

**Que introduzimos no método ```main()```**:

- ```fileServer: = http.FileServer(http.Dir ("static"))```: Aqui, criamos um servidor de arquivos usando o manipulador FileServer do pacote ```net/http```, que atende a solicitações HTTP do diretório estático presente no sistema de arquivos.

- ```http.Handle("/ static /", http.StripPrefix ("/ static /", fileServer))```: Aqui, estamos registrando o manipulador ```http.StripPrefix ("/ static/", fileServer)``` com o padrão de URL ```/static``` usando ```HandleFunc``` do pacote ```net/http```, que significa que ```http.StripPrefix ("/ static/", fileServer)``` é executado e passa ```(http.ResponseWriter, * http.Request)``` como um parâmetro para ele sempre que acessar a URL HTTP com o ```/static```.

- ```http.StripPrefix("/ static /", fileServer)```: retorna um manipulador que atende a solicitações HTTP removendo ```/static``` do caminho da URL da solicitação e chama o servidor de arquivos. O ```StripPrefix``` manipula uma solicitação para um caminho que não começa com um prefixo respondendo com um ```HTTP 404```.


## Servindo arquivos estáticos através de HTTP usando o Gorilla Mux (Serving static files over HTTP using Gorilla Mux)
Servimos recursos estáticos por meio do servidor de arquivos HTTP da Go. Nessa parte, veremos como podemos atendê-lo através do roteador Gorilla Mux, que também é uma das formas mais comuns de criar um roteador HTTP.

Como já criamos um template que serve ```main.css``` do diretório ```static/css``` presente no sistema de arquivos em nossa receita anterior, vamos apenas atualizá-lo para usar o roteador Gorilla Mux.

- 1. Instalação ```github.com/gorilla/mux``` usando o comando ```go get```:

```$ go get github.com/gorilla/mux```

- 2.Crie ```serve-static-files-gorilla-mux.go```, onde vamos criar um roteador Gorilla Mux ao invés de um HTTP ```FileServer```:

```
package main
import 
(
  "html/template"
  "log"
  "net/http"
  "github.com/gorilla/mux"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type Person struct 
{
  Id string
  Name string
}
func renderTemplate(w http.ResponseWriter, r *http.Request) 
{
  person := Person{Id: "1", Name: "Foo"}
  parsedTemplate, _ := template.ParseFiles("templates/
  first-template.html")
  err := parsedTemplate.Execute(w, person)
  if err != nil 
  {
    log.Printf("Error occurred while executing the template 
    or writing its output : ", err)
    return
  }
}
func main() 
{
  router := mux.NewRouter()
  router.HandleFunc("/", renderTemplate).Methods("GET")
  router.PathPrefix("/").Handler(http.StripPrefix("/static",
  http.FileServer(http.Dir("static/"))))
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, router)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.Rode o programa no terminal:

```$ go run serve-static-files-gorilla-mux.go```


### Como isso deve funcionar
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

Navegar ```http://localhost:8080``` nos mostrará a mesma saída que vimos anteriormente.

**Entendo as partes**

- ```router: = mux.NewRouter()```: Aqui nós instanciamos o roteador ```gorilla/mux``` chamando o manipulador ```NewRouter()``` do roteador mux.

- ```router.HandleFunc("/", renderTemplate).Methods("GET")```: Aqui registramos o padrão / URL com o manipulador ```renderTemplate```. Isso significa que o ```renderTemplate``` será executado para cada solicitação com o padrão de URL /.

- ```router.PathPrefix("/").Handler(http.StripPrefix("/ estático", http.FileServer (http.Dir ("static /"))))```: Aqui estamos nos registrando / como uma nova rota, juntamente com a configuração do manipulador a ser executado uma vez que é chamado.

- ```http.StripPrefix ("/ static", http.FileServer (http.Dir ("static /")))```: Isso retorna um manipulador que atende solicitações HTTP removendo / static do caminho da URL da solicitação e chamando o servidor de arquivos. O StripPrefix manipula uma solicitação para um caminho que não começa com um prefixo respondendo com um HTTP 404.


## Criando seu primeiro HTML form
Sempre que quisermos coletar os dados do cliente e enviá-los ao servidor para processamento, a implementação de um formulário HTML é a melhor escolha. 

Criaremos um formulário HTML simples que possui dois campos de entrada e um botão para enviar o formulário. Faça o seguinte:

- 1.Crie ```login-form.html``` dentro do diretório de modelos (ou templates), da seguinte maneira:

```$ mkdir templates && cd templates && touch login-form.html```

- 2.Copie o seguinte código

```
<html>
  <head>
    <title>First Form</title>
  </head>
  <body>
    <h1>Login</h1>
    <form method="post" action="/login">
      <label for="username">Username</label>
      <input type="text" id="username" name="username">
      <label for="password">Password</label>
      <input type="password" id="password" name="password">
      <button type="submit">Login</button>
    </form>
  </body>
</html>
```

> O modelo anterior tem duas caixas de texto ```username``` e ```password``` junto com um botão Login.

> Ao clicar no botão Login, o cliente fará uma chamada ```POST``` para uma ação definida em um formulário HTML, que é ```/login``` no nosso caso.

- 3.Crie ```html-form.go```, onde analisaremos o modelo de formulário e o escreveremos em um fluxo de resposta HTTP, da seguinte maneira:

```
package main
import 
(
  "html/template"
  "log"
  "net/http"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
func login(w http.ResponseWriter, r *http.Request) 
{
  parsedTemplate, _ := template.ParseFiles("templates/
  login-form.html")
  parsedTemplate.Execute(w, nil)
}
func main() 
{
  http.HandleFunc("/", login)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 4.Rode o programa no terminal:

```$ go run html-form.go```


### Como isso deve funcionar
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta 8080. A navegação em http: // localhost: 8080 nos mostrará um formulário em HTML.

**Entendo as partes**

- ```func login (w http.ResponseWriter, r * http.Request) {parsedTemplate, _: = template.ParseFiles ("modelos / login-form.html") parsedTemplate.Execute (w, nil)}```: Esta é uma função Go que aceita ```ResponseWriter``` e ```Request``` como parâmetros de entrada, analisa ```login-form.html``` e retorna um novo modelo.

- ```http.HandleFunc ("/", login)```: Aqui estamos registrando uma função de login com o padrão / URL usando ```HandleFunc``` do pacote ```net/http```, o que significa que a função de login é executada toda vez que acessamos a URL HTTP com o / pattern passando ```ResponseWriter``` e ```Request``` como os parâmetros para isso.

- ```err: = http.ListenAndServe (CONN_HOST + ":" + CONN_PORT, nil)```: Aqui estamos chamando ```http.ListenAndServe``` para atender a solicitações HTTP que tratam cada conexão de entrada em um Goroutine separado. O ```ListenAndServe``` aceita dois parâmetros - o endereço do servidor e o manipulador - em que o endereço do servidor é ```localhost:8080``` e o manipulador é ```nil```.

- ```if err! = nil {log.Fatal ("erro ao iniciar servidor http:", err) return}```: Aqui nós verificamos se há um problema com o início do servidor. Se houver, registre o erro e saia com um código de status 1.


## Lendo o seu primeiro HTML form
Depois que um formulário HTML é enviado, precisamos ler os dados do cliente no lado do servidor para tomar uma ação apropriada.

Como já criamos um formulário HTML no passo anterior, vamos apenas ler seus valores de campo.

- 1.Instale o ```github.com/gorilla/schema``` usando o comando ```go get```

```$ go get github.com/gorilla/schema```

- 2.Crie ```html-form-read.go```, onde leremos um campo de formulário HTML depois de decodificá-lo usando o pacote ```github.com/gorilla/schema``` e escreveremos Hello seguido do nome de usuário para um fluxo de response HTTP

```
package main
import 
(
  "fmt"
  "html/template"
  "log"
  "net/http"
  "github.com/gorilla/schema"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
type User struct 
{
  Username string
  Password string
}
func readForm(r *http.Request) *User 
{
  r.ParseForm()
  user := new(User)
  decoder := schema.NewDecoder()
  decodeErr := decoder.Decode(user, r.PostForm)
  if decodeErr != nil 
  {
    log.Printf("error mapping parsed form data to struct : ",
    decodeErr)
  }
  return user
}
func login(w http.ResponseWriter, r *http.Request) 
{
  if r.Method == "GET" 
  {
    parsedTemplate, _ := template.ParseFiles("templates/
    login-form.html")
    parsedTemplate.Execute(w, nil)
  } 
  else 
  {
    user := readForm(r)
    fmt.Fprintf(w, "Hello "+user.Username+"!")
  }
}
func main() 
{
  http.HandleFunc("/", login)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.Rode o programa no terminal:

```$ go run html-form-read.go```

### Como isso deve funcionar
Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```. 
Navegando ```http://localhost:8080``` nos mostrará um formulário em HTML.

Assim que digitarmos o nome de usuário e a senha e clicarmos no botão Login, veremos Hello seguido do nome de usuário como resposta do servidor.

**Entendendo as partes**
- 1.Usando ```import ("fmt" "html / template" "log" "net / http" "github.com/gorilla/schema")```, nós importamos dois pacotes adicionais ```fmt``` e ```github.com/gorilla/schema``` - que ajudam a converter ```structs``` de e para ```forms``` valores. 

- 2.Definimos o tipo ```User struct``` que possui campos de ```nome de usuário``` | ```Username```  e ```senha``` | ```Password```, da seguinte maneira:

```
type User struct 
{
  Username string
  Password string
}
```

- 3.Em seguida, definimos o manipulador ```readForm```, que recebe a solicitação ```HTTP request``` como um parâmetro de entrada e retorna o usuário da seguinte maneira:

```
func readForm(r *http.Request) *User {
 r.ParseForm()
 user := new(User)
 decoder := schema.NewDecoder()
 decodeErr := decoder.Decode(user, r.PostForm)
 if decodeErr != nil {
 log.Printf("error mapping parsed form data to struct : ", decodeErr)
 }
 return user
 }
```

**Entendo as partes**
- ```r.ParseForm ()```: Aqui analisamos o corpo da solicitação como um formulário e colocamos os resultados em ```r.PostForm``` e ```r.Form```.

- ```user: = new (User)```: Aqui criamos um novo tipo de ```user struct```.

- decoder: = schema.NewDecoder (): Aqui estamos criando um decodificador, o qual estaremos usando para preencher um ```struct``` de usuário com valores ```Form```. (```user struct``` | ```form values```)

- ```decodeErr: = decoder.Decode (user, r.PostForm)```: Aqui nós decodificamos dados de formulários parseados dos parâmetros do corpo do ```POST``` para um ```struct``` do usuário (```user struct).
> r.PostForm is only available after ParseForm is called.

- ```if decodeErr != nil { log.Printf("error mapping parsed form data to struct : ", decodeErr) }```: Aqui nós verificamos se existe algum problema com dados de formulário de mapeamento em uma estrutura. Se houver, registre-o.

Em seguida, definimos um manipulador de login, que verifica se a solicitação HTTP chamando o manipulador é uma solicitação GET e, em seguida, analisa login-form.html no diretório de modelos e grava-o em um fluxo de resposta HTTP; Caso contrário, ele chama o manipulador readForm, da seguinte maneira:


```
func login(w http.ResponseWriter, r *http.Request) 
{
  if r.Method == "GET" 
  {
    parsedTemplate, _ := template.ParseFiles("templates/
    login-form.html")
    parsedTemplate.Execute(w, nil)
  } 
  else 
  {
    user := readForm(r)
    fmt.Fprintf(w, "Hello "+user.Username+"!")
  }
}
```

## Validando o HTML form
Na maioria das vezes, temos que validar a entrada de um cliente antes de processá-lo, o que pode ser obtido através do número de pacotes externos no Go, como ```gopkg.in/go-playground/validator.v9, gopkg.in/validator.v2``` e ```github.com/asaskevich/govalidator```.

Trabalharemos com o validador mais famoso e comumente usado, ```github.com/asaskevich/govalidator```, para validar nosso formulário HTML.

Como já criamos e lemos um formulário HTML, vamos apenas validar seus valores de campo.

- 1.Instale ```github.com/asaskevich/govalidator``` e o pacote ```github.com/gorilla/schema usando``` o comando go get:

```
$ go get github.com/asaskevich/govalidator
$ go get github.com/gorilla/schema
```

- 2.Crie ```html-form-validation.go```, onde vamos ler um formulário HTML, decodificá-lo usando ```github.com/gorilla/schema``` e validar cada campo dele em uma tag definida ```User struct``` usando ```github.com/asaskevich/govalidator```:

```
package main
import 
(
  "fmt"
  "html/template"
  "log"
  "net/http"
  "github.com/asaskevich/govalidator"
  "github.com/gorilla/schema"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
  USERNAME_ERROR_MESSAGE = "Please enter a valid Username"
  PASSWORD_ERROR_MESSAGE = "Please enter a valid Password"
  GENERIC_ERROR_MESSAGE = "Validation Error"
)
type User struct 
{
  Username string `valid:"alpha,required"`
  Password string `valid:"alpha,required"`
}
func readForm(r *http.Request) *User 
{
  r.ParseForm()
  user := new(User)
  decoder := schema.NewDecoder()
  decodeErr := decoder.Decode(user, r.PostForm)
  if decodeErr != nil 
  {
    log.Printf("error mapping parsed form data to struct : ",
    decodeErr)
  }
  return user
}
func validateUser(w http.ResponseWriter, r *http.Request, user *User) (bool, string) 
{
  valid, validationError := govalidator.ValidateStruct(user)
  if !valid 
  {
    usernameError := govalidator.ErrorByField(validationError,
    "Username")
    passwordError := govalidator.ErrorByField(validationError,
    "Password")
    if usernameError != "" 
    {
      log.Printf("username validation error : ", usernameError)
      return valid, USERNAME_ERROR_MESSAGE
    }
    if passwordError != "" 
    {
      log.Printf("password validation error : ", passwordError)
      return valid, PASSWORD_ERROR_MESSAGE
    }
  }
  return valid, GENERIC_ERROR_MESSAGE
}
func login(w http.ResponseWriter, r *http.Request) 
{
  if r.Method == "GET" 
  {
    parsedTemplate, _ := template.ParseFiles("templates/
    login-form.html")
    parsedTemplate.Execute(w, nil)
  } 
  else 
  {
    user := readForm(r)
    valid, validationErrorMessage := validateUser(w, r, user)
    if !valid 
    {
      fmt.Fprintf(w, validationErrorMessage)
      return
    }
    fmt.Fprintf(w, "Hello "+user.Username+"!")
  }
}
func main() 
{
  http.HandleFunc("/", login)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 3.Rode no terminal:

```$ go run html-form-validation.go```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta 8080. 

- Navegando ```http://localhost:8080``` nos mostrará um formulário em HTML

- Em seguida, envie o formulário com os valores válidos

- Nos mostrará o Hello seguido do nome de usuário na tela do navegador

- O envio do formulário com o valor como não-alfa em qualquer um dos campos mostrará a mensagem de erro. Por exemplo, enviando o formulário com o valor Username como 1234

- Nos mostrará uma mensagem de erro no navegador

- Além disso, podemos enviar um formulário HTML a partir da linha de comando como:

```$ curl --data "username=Foo&password=password" http://localhost:8080/```


**Entendendo as partes** 
- Usando ```import ("fmt", "html/template", "log", "net/http" "github.com/asaskevich/govalidator" "github.com/gorilla/schema")```, nós importamos um pacote adicional - o ```github.com/asaskevich/govalidator```, que nos ajuda a validar as estruturas.

- Em seguida, atualizamos o tipo ```User struct``` para incluir uma tag literal de string com a ```key``` como ```valid``` e ```value``` como ```alpha```, obrigatório, da seguinte forma:

```
type User struct 
{
  Username string `valid:"alpha,required"`
  Password string 
  valid:"alpha,required"
}
```

- Em seguida, definimos um manipulador ```validateUser```, que recebe ```ResponseWriter```, ```Request``` e ```User``` como entradas e retorna um ```bool``` e uma ```string```, que são a mensagem de erro de validação e status struct respectivamente. Neste manipulador, validamos as tags de estrutura chamando o manipulador ```ValidateStruct``` do ```govalidator```. Se houver um erro na validação do campo, buscaremos o erro que chama o manipulador ```ErrorByField``` do ```govalidator``` e retornamos o resultado junto com a mensagem de erro de validação.

- Em seguida, atualizamos o manipulador de ```login``` para chamar ```validateUser``` passing ```(wResp.ResponseWriter, r * http.Request, user * User)``` como parâmetros de entrada para ele e verificar se há algum erro de validação. Se houver erros, gravamos uma mensagem de erro em um fluxo de resposta HTTP e o retornamos.


## Uploading files
Um dos cenários mais comuns em qualquer aplicativo da Web é enviar um arquivo ou uma pasta para o servidor. Por exemplo, se estivermos desenvolvendo um portal de emprego, talvez tenhamos que fornecer uma opção em que o candidato possa fazer o upload de seu perfil/currículo ou, digamos, desenvolver um site de comércio eletrônico com um recurso em que o cliente possa faça o upload de seus pedidos em massa usando um arquivo.

Conseguir a funcionalidade de fazer o upload de um arquivo no Go é bastante fácil usando seus pacotes integrados.

Vamos criar um formulário HTML com um campo de tipo de arquivo, que permite ao usuário escolher um ou mais arquivos para enviar para um servidor por meio de um envio de formulário. Execute os seguintes passos:

- 1.Crie ```upload-file.html``` dentro do diretório de modelos


```$ mkdir templates && cd templates && touch upload-file.html```


- 2.Copie o seguinte conteúdo para ```upload-file.html```:


```
<html>
  <head>
    <meta charset="utf-8">
    <title>File Upload</title>
  </head>
  <body>
    <form action="/upload" method="post" enctype="multipart/
    form-data">
      <label for="file">File:</label>
      <input type="file" name="file" id="file">
      <input type="submit" name="submit" value="Submit">
    </form>
  </body>
</html>
```

> No modelo anterior, definimos um campo do tipo file junto com um botão ```Submit```.

> Ao clicar no botão Enviar, o cliente codifica os dados que formam o corpo da solicitação e faz uma chamada POST para a ação do formulário, que é 
```/upload``` no nosso caso.

- 3.Crie ```upload-file.go```, onde definiremos manipuladores para renderizar o modelo de upload de arquivo, obter o arquivo da solicitação, processá-lo e gravar a resposta em um fluxo de resposta HTTP, da seguinte maneira:

```
package main
import 
(
  "fmt"
  "html/template"
  "io"
  "log"
  "net/http"
  "os"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
func fileHandler(w http.ResponseWriter, r *http.Request) 
{
  file, header, err := r.FormFile("file")
  if err != nil 
  {
    log.Printf("error getting a file for the provided form key : ",
    err)
    return
  }
  defer file.Close()
  out, pathError := os.Create("/tmp/uploadedFile")
  if pathError != nil 
  {
    log.Printf("error creating a file for writing : ", pathError)
    return
  }
  defer out.Close()
  _, copyFileError := io.Copy(out, file)
  if copyFileError != nil 
  {
    log.Printf("error occurred while file copy : ", copyFileError)
  }
  fmt.Fprintf(w, "File uploaded successfully : "+header.Filename)
}
func index(w http.ResponseWriter, r *http.Request) 
{
  parsedTemplate, _ := template.ParseFiles("templates/
  upload-file.html")
  parsedTemplate.Execute(w, nil)
}
func main() 
{
  http.HandleFunc("/", index)
  http.HandleFunc("/upload", fileHandler)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}
```

- 4.Rode o programa no terminal

```$ go run upload-file.go```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```. A navegação em ```http://localhost:8080``` nos mostrará o Formulário de Upload de Arquivos

- Pressionar o botão Enviar depois de escolher um arquivo resultará na criação de um arquivo no servidor com o nome ```uploadFile``` dentro do diretório ```/tmp```.

- O upload bem-sucedido exibirá a mensagem no navegador

**Entendendo as partes**
Definimos o manipulador ```fileHandler()```, que obtém o arquivo da solicitação, lê seu conteúdo e, eventualmente, o grava em um arquivo em um servidor. Como esse manipulador faz muitas coisas, vamos olhar detalhadamente:

- ```file, header, err: = r.FormFile ("file")```: Aqui chamamos o manipulador FormFile na solicitação HTTP para obter o arquivo para a chave de formulário fornecida.

- ```if err! = nil {log.Printf ("erro ao obter um arquivo para a chave de formulário fornecida:", err) return}```: Aqui nós verificamos se há algum problema ao obter o arquivo da requisição. Se houver, registre o erro e saia com um código de status 1.

- ```defer file.Close()```: A instrução defere fecha o arquivo uma vez que retornamos da função.


- ```out, pathError: = os.Create("/ tmp / uploadedFile")```: Aqui estamos criando um arquivo chamado uploadedFile dentro de um diretório ```/tmp``` com o modo 666, o que significa que o cliente pode ler e escrever, mas não pode executar o arquivo.

- ```if pathError! = nil {log.Printf ("erro ao criar um arquivo para escrever:", pathError) return}```: Aqui nós verificamos se há algum problema com a criação de um arquivo no servidor. Se houver, registre o erro e saia com um código de status 1.

- ```_, copyFileError: = io.Copy (out, file)```: Aqui copiamos o conteúdo do arquivo que recebemos para o arquivo que criamos dentro do diretório ```/tmp```.

- ```fmt.Fprintf (w, "File uploaded successfully:" + header.Filename)```: Aqui escrevemos uma mensagem junto com um nome de arquivo para um fluxo de resposta HTTP.


















