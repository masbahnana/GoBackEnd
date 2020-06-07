# Templates, Static Filles e HTML Forms
Muitas vezes, gostaríamos de criar formulários HTML para obter as informações de um cliente em um formato especificado, fazer upload de arquivos ou pastas para o servidor e gerar modelos HTML genéricos, em vez de repetir o mesmo texto estático. Com o conhecimento dos conceitos abordados neste capítulo, seremos capazes de implementar todas essas funcionalidades eficientemente no Go.Neste capítulo, começaremos criando um template básico e depois passaremos a servir arquivos estáticos, como .js, .css e imagens de um sistema de arquivos e, eventualmente, criar, ler e validar formulários HTML e carregar um arquivo no servidor.

## Criando o primeiro template
Os templates nos permitem definir espaços reservados para conteúdo dinâmico que podem ser substituídos pelos valores em tempo de execução por um mecanismo de modelo/template. Eles podem então ser transformados em um arquivo HTML e enviados para o cliente. Criar modelos no Go é bastante fácil usando o pacote ```html/template``` de Go.

Vamos criar um primeiro template.html com alguns espaços reservados cujo valor será injetado pelo mecanismo de modelo no tempo de execução. Execute os seguintes passos:

- 1.Crie ```first-template.html``` dentro do diretório templates (crie esse diretório também), executando o comando:

```bash
$ mkdir templates && cd templates && touch first-template.html
```

- 2.Copie o código no ```first-template.html```:

```html
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

```go
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

```bash
$ go run first-template.go
```


### Como isso funciona

Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta ```8080```.

Navegando ```http://localhost:8080``` nos mostrará o ```Hello Foo!``` que está no modelo

Execute ```curl -X GET http://localhost:8080``` na linha de comando:

```bash
$ curl -X GET http://localhost:8080
```

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
Ao projetar aplicativos da web, é sempre uma prática recomendada servir recursos estáticos, como ".js", ".css" e "images", do sistema de arquivos ou de qualquer **rede de entrega de conteúdo (CDN - content delivery network)**, como Akamai ou Amazon CloudFront, em vez de servi-los o servidor da web. Isso ocorre porque todos esses tipos de arquivos são estáticos e não precisam ser processados; Então, por que devemos colocar carga extra no servidor? Além disso, ajuda a aumentar o desempenho do aplicativo, pois todas as solicitações para os arquivos estáticos serão fornecidas por fontes externas e, portanto, reduzirão a carga no servidor.

O pacote net / http do Go é suficiente para servir recursos estáticos do sistema de arquivos através do FileServer, que será abordado nesta receita.


